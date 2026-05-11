# bitio-kern Tasks

`bitio-kern` should become the default Kern package for bit-level protocol and
codec work. The package replaces the old incubator experiment with a small,
auditable, fluent API built around borrowed buffers and explicit cursor
handles.

## Priority 0: Package Shape

- Create a standalone Craft package named `bitio` with `src/lib.rn`, focused
  tests, README, MIT license, and no dependency on the old incubator source.
- Keep the public API allocation-free by default. Bit readers and writers should
  borrow caller-owned byte slices and report cursor/buffer errors explicitly.
- Use handle-style APIs:
  - `bytes.bits()` creates a `BitReader`.
  - `out.bits_mut()` creates a `BitWriter`.
  - `reader.read_u8(width)`, `reader.read_bool()`, and
    `reader.align_to_byte()` advance the reader.
  - `writer.write_u8(width, value)`, `writer.write_bool(value)`, and
    `writer.pad_to_byte()` advance the writer.
- Keep module-level functions for constructors and stateless byte-order helpers;
  once a reader/writer exists, the handle should be the action surface.

## Priority 1: Core Bit Cursors

- Implement MSB-first and LSB-first reader/writer modes as explicit cursor
  types or explicit constructors. Do not hide bit order in a boolean flag.
- Track byte offset and bit offset in the cursor and expose position methods for
  diagnostics and protocol parsers.
- Return structured errors:
  - `UnexpectedEof`
  - `ShortOutput`
  - `InvalidBitCount`
  - `ValueTooWide`
- Support fixed-width reads and writes for `u8`, `u16`, `u32`, `u64`, and
  `usize` where width constraints are clear.

## Priority 2: Byte Order And Masks

- Provide fluent byte-order helpers on borrowed bytes and mutable byte slices:
  - `bytes.read_u16_be()`, `bytes.read_u32_le()`, etc.
  - `out.write_u32_be(value)`, `out.write_u64_le(value)`, etc.
- Keep short-buffer handling structured rather than returning sentinel values.
- Provide XOR mask helpers for protocol buffers without exposing SIMD-specific
  types in the public API.

## Priority 3: Documentation And Tests

- README should show a real packet parser/writer flow with an `app()` function
  that propagates domain errors.
- Module docs should explain bit order, cursor ownership, byte alignment, and
  error behavior.
- Public items need `///` comments for cursor movement, buffer ownership, and
  failure cases.
- Tests must cover:
  - MSB-first and LSB-first reads/writes
  - cross-byte reads and writes
  - byte alignment and zero padding
  - short input/output errors
  - invalid widths and value-too-wide failures
  - endian helpers

## Priority 4: Expansion Path

- Signed integer helpers with explicit sign-extension behavior.
- Varint helpers for common protocols.
- Stream adapters when Kern has a stronger async or IO story.
- Optional generated parser examples for protocol documentation.

## Done For First Publishable Version

- `craft fmt --check --verbose --color never` passes.
- `craft test --color never` passes.
- `craft style --verbose --color never` reports no missing public docs for the
  hand-written public API.
- README examples have matching compile-only tests.
- No compatibility wrapper is kept for incubator-era names.
