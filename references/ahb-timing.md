# AHB Timing Reference

Use this reference for AMBA AHB and AHB-Lite transfer timing. It focuses on the
protocol rules needed to draw an accurate waveform rather than reproducing the
full AMBA specification.

## Version Scope

- AHB-Lite is the single-master subset of AHB. It keeps the address, control,
  data, and response signals but omits arbitration signals such as `HBUSREQ`,
  `HGRANT`, `HLOCK`, and `HMASTER`.
- AHB adds split and retry responses through `HRESP`, plus arbitration.
- Include arbitration or split/retry signals only when the requested interface
  uses them. Do not imply that a basic AHB-Lite transfer has a master select or
  arbitration phase.

## Address Phase and Data Phase

Every transfer is pipelined: the address phase of one transfer overlaps the
data phase of the previous transfer.

1. **Address phase**: present `HADDR`, `HTRANS`, `HWRITE`, `HSIZE`, `HBURST`,
   and optional `HPROT` for one period.
2. **Data phase**: one period later, drive `HWDATA` for a write or sample
   `HRDATA` for a read.
3. `HRESP` is valid with the data phase; `OKAY` is the normal response.

Draw the address and data lanes in separate groups so the one-period pipeline
offset between them is visible.

## Wait States

`HREADY` extends both the data phase and the address phase:

- When `HREADY` is low, the current transfer has not completed. Hold the
  address, control, write data, and read data stable by extending them with `.`.
- Complete the transfer only on the rising edge where `HREADY` is high.
- A wait state does not start a new transfer and does not change `HTRANS` from
  its current value.

## Burst Types

`HBURST` selects the transfer type. `HTRANS` is `NONSEQ` for the first transfer
of a burst and `SEQ` for the remaining transfers.

| `HBURST` | Type | Address behaviour |
|----------|------|-------------------|
| `SINGLE` | Single transfer | One beat, `HTRANS` is `NONSEQ` |
| `INCR` | Unspecified-length increment | Address increments, no wrap |
| `WRAP4` / `WRAP8` / `WRAP16` | Wrapping burst of 4/8/16 beats | Address wraps at a size-aligned boundary |
| `INCR4` / `INCR8` / `INCR16` | Fixed-length incrementing burst | Address increments, does not wrap |

### Wrapping Burst

- A `WRAPn` burst wraps when the address crosses the boundary of the burst
  size in bytes (beats x `HSIZE` width).
- The start address must be aligned to the total burst size.
- A four-beat word (`HSIZE` = Word) burst starting at `0x38` steps `0x38`,
  `0x3C`, then wraps to `0x30`, `0x34`, because `0x3C` is the top of the
  16-byte aligned block.

## Signal Stability

- Hold `HADDR` and control signals stable through wait states using `.`.
- Write data is driven with the data phase and held while `HREADY` is low.
- Read data must be valid during the data phase where the transfer completes.
- A blank bus segment before and after the burst keeps the data lane in
  double-line bus form during idle.

## Drawing Conventions

- Include `HCLK` and use one character per AHB clock period.
- Use `phase: -0.5` consistently on non-clock lanes in the provided examples.
- Keep address and data lanes in continuous bus form, including empty-labeled
  bus segments before and after the transfer.
- Use `.` to hold every stable value, especially through wait states.
- Address and control lanes may start a new bus segment at each beat boundary,
  aligned with the `HADDR` change, even when `HTRANS` stays `SEQ`. The
  crossover marks a beat boundary that is consistent with the address and data
  change, not a spurious value transition.
- Label a completion or wrap condition in the footer when waits or wrapping are
  central to the scenario.

## Example Selection

| Scenario | Example |
|----------|---------|
| Single write followed by single read | `examples/ahb_lite_read_write.json` |
| Four-beat wrapping burst with a wait state | `examples/ahb_four_beat_wrapping_burst.json` |

## Common Errors

- Merging the address and data phases into a single lane or group.
- Changing `HADDR` or control during a wait state instead of holding it.
- Drawing a wrapping burst as a linear increment across the boundary.
- Treating each `SEQ` beat as a new transfer instead of the next beat of the
  same burst.
- Dropping the bus into a single-bit line during idle instead of keeping an
  empty-labeled bus segment.
- Sampling `HRDATA` before the completion edge where `HREADY` is high.
