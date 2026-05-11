# bitio-kern

Bit-level protocol and codec utilities for Kern.

The Craft package name is `bitio`; the repository name is `bitio-kern`.
The package is built around borrowed byte buffers, explicit cursor handles, and
structured cursor errors. Ordinary reads, writes, endian helpers, and masks do
not allocate.

## Usage

```toml
[dependencies]
bitio = { git = "https://github.com/softfault/bitio-kern.git" }
```

Local development can use a path dependency:

```toml
[dependencies]
bitio = { path = "../bitio-kern" }
```

```kern
use bitio;

struct Header {
    encrypted: bool,
    version: u8,
    length: u16,
};

fn parse_header(packet: &[u8]) Header!bitio.Error {
    let r = packet.bits_msb()..&;
    let encrypted = r.read_bool().!;
    let version = r.read_u8(3).!;
    r.align_to_byte();

    let length = packet.&[1 .. #packet].read_u16_be().!;
    return .{ Ok: .{ encrypted: encrypted, version: version, length: length } };
}

fn write_header(out: &mut [u8], header: Header) void!bitio.Error {
    let w = out.bits_msb_mut()..&;
    w.write_bool(header.encrypted).!;
    w.write_u8(3, header.version).!;
    w.pad_to_byte().!;
    out..&[1 .. #out].write_u16_be(header.length).!;
    return .{ Ok: {} };
}
```

## API Shape

Bit order is explicit in the cursor type:

- `bytes.bits_msb()` creates an `MsbReader`.
- `bytes.bits_lsb()` creates an `LsbReader`.
- `out.bits_msb_mut()` creates an `MsbWriter`.
- `out.bits_lsb_mut()` creates an `LsbWriter`.

Readers expose `read_bool`, `read_u8`, `read_u16`, `read_u32`, `read_u64`,
`read_usize`, `align_to_byte`, and cursor position queries. Writers expose the
matching `write_*` methods plus `pad_to_byte`.

Borrowed byte slices expose endian reads such as `read_u16_be` and
`read_u64_le`. Mutable byte slices expose endian writes plus `xor_mask` and
`xor_repeating` for protocol buffers.

## Errors

All fallible operations return `T!bitio.Error` or `void!bitio.Error`:

- `UnexpectedEof`: input ended before the requested bits or bytes were present.
- `ShortOutput`: output ended before the requested bits or bytes fit.
- `InvalidBitCount`: the requested bit width is larger than the destination type.
- `ValueTooWide`: a writer value does not fit in the requested bit width.

## License

`bitio-kern` is distributed under the MIT License. See `LICENSE`.
