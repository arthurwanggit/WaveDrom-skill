# SPI NOR Flash Read Timing

Use these notes with the three `examples/spi_*_read.json` files. They are
WaveDrom modeling references, not a substitute for a target Flash datasheet.

## Example Transactions

| Example | Command | Widths | Address | Data |
|---------|---------|--------|---------|------|
| Single read | `0x03` | 1-1-1 | Serial, MSB first | Serial |
| Dual I/O read | `0xBB` | 1-2-2 | Two bits per clock | Two bits per clock |
| Quad I/O read | `0xEB` | 1-4-4 | Four bits per clock | Four bits per clock |

The Dual and Quad examples intentionally show 8 dummy clocks and no mode-bit
phase. Dummy count, mode bits, alternate bytes, command values, and continuous
read behavior are device-dependent. Never add or remove those fields without
an explicit requirement or the target device specification.

## Drawing Rules

- Treat CMD, ADDR, dummy, and DATA as semantic phases. Each appears once on the
  Phase summary lane and uses one colored bus segment and one label.
- A timeline gap compresses detailed bit or clock activity; it is not a phase
  boundary. Keep the Phase lane continuous with `.` at each compressed slot.
- Put `|` on SCLK, CS#, and relevant IO lanes so the omitted interval remains
  visible and aligned.
- Label address as one `ADDR` phase. Terms such as `ADDR high` and `ADDR low`
  describe visible fragments, not separate SPI protocol phases.
- Use one color per semantic group across the Phase and IO lanes. The examples
  use `3` for CMD, `4` for ADDR, `6` for dummy, and `7` for DATA.
- State every omission in the footnote, including exact bit ranges, byte
  ranges, or clock counts.
- Preserve MSB-first lane mapping. For Quad address clock 1, for example, IO3
  carries A23 and IO0 carries A20.

## Compression Convention

These examples show no more than six detailed clocks for a long interval when
that is sufficient: three clocks before the gap and three after it. The Phase
lane spans the compressed position without a crossover. Use more visible
clocks when fewer would make the beginning or ending behavior ambiguous.
