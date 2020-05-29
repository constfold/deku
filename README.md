# Deku

[![Latest Version](https://img.shields.io/crates/v/deku.svg)](https://crates.io/crates/deku)
[![Rust Documentation](https://docs.rs/deku/badge.svg)](https://docs.rs/deku)
[![Actions Status](https://github.com/sharksforarms/deku/workflows/CI/badge.svg)](https://github.com/sharksforarms/deku/actions)
[![codecov](https://codecov.io/gh/sharksforarms/deku/branch/master/graph/badge.svg)](https://codecov.io/gh/sharksforarms/deku)

Declaritive binary reading and writing

This crate provides bit-level, symetric, serialization/deserialization implementations for structs and enums

## Why use Deku

**Productivity**: Focus and declare your types that represent the data, Deku will generate symetric reader/writer functions for your type! Avoid the requirement of writing redundant, error-prone parsing and writing code for binary structs or network headers

## Usage

```toml
[dependencies]
deku = "0.1"
```

## Example

See [examples](https://github.com/sharksforarms/deku/tree/master/examples) for more!

```rust
use deku::prelude::*;
use std::convert::TryFrom;

#[derive(Debug, PartialEq, DekuRead, DekuWrite)]
struct FieldF {
    #[deku(bits = "6")]
    data: u8,
}

/// DekuTest Struct
//   0                   1                   2                   3                   4
//   0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0
//  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
//  |    field_a    |   field_b   |c|            field_d              | e |     f     |
//  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
//
#[derive(Debug, PartialEq, DekuRead, DekuWrite)]
// #[deku(endian = "little")] // By default it uses the system endianess, but can be overwritten
struct DekuTest {
    field_a: u8,
    #[deku(bits = "7")]
    field_b: u8,
    #[deku(bits = "1")]
    field_c: u8,
    #[deku(endian = "big")]
    field_d: u16,
    #[deku(bits = "2")]
    field_e: u8,
    field_f: FieldF,
    num_items: u8,
    #[deku(len = "num_items", endian = "big")]
    items: Vec<u16>,
}

fn main() {
    let test_data: &[u8] = [
        0xAB,
        0b1010010_1,
        0xAB,
        0xCD,
        0b1100_0110,
        0x02,
        0xBE,
        0xEF,
        0xC0,
        0xFE,
    ]
    .as_ref();

    let test_deku = DekuTest::try_from(test_data).unwrap();

    assert_eq!(
        DekuTest {
            field_a: 0xAB,
            field_b: 0b0_1010010,
            field_c: 0b0000000_1,
            field_d: 0xABCD,
            field_e: 0b0000_0011,
            field_f: FieldF { data: 0b00_000110 },
            num_items: 2,
            items: vec![0xBEEF, 0xC0FE],
        },
        test_deku
    );

    let test_deku: Vec<u8> = test_deku.into();
    assert_eq!(test_data.to_vec(), test_deku);
}
```
