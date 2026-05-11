# bitio-kern

Bit-level protocol and codec utilities for Kern.

The Craft package name is `bitio`; the repository name is `bitio-kern`.
The package is designed around borrowed byte buffers, explicit cursor handles,
and structured cursor errors.

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

fn read_flags(packet: &[u8]) bool!bitio.Error {
    let r = packet.bits_msb()..&;
    let enabled = r.read_bool().!;
    _ = r.read_u8(3).!;
    return .{ Ok: enabled };
}
```

Planned entry points:

- `bytes.bits_msb()` and `bytes.bits_lsb()` create borrowed bit readers.
- `out.bits_msb_mut()` and `out.bits_lsb_mut()` create borrowed bit writers.
- Reader and writer handles expose cursor movement and byte alignment methods.
- Borrowed byte slices expose endian read helpers.
- Mutable byte slices expose endian write helpers and masking helpers.

## Design

`bitio-kern` does not allocate for ordinary cursor operations. The caller owns
input and output buffers, and cursor methods report structured errors for short
input, short output, invalid widths, and values that do not fit the requested
bit count.

The first implementation is intentionally byte-oriented and protocol-oriented.
It does not expose SIMD or volatile-memory concepts in the public API.

## License

`bitio-kern` is distributed under the MIT License. See `LICENSE`.
