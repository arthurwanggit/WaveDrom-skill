---
name: wavedrom-timing-diagram
description: Use when generating or editing WaveDrom timing diagrams for digital circuits, bus protocols, RTL signal definitions, or any waveform visualization. Do not use for non-WaveDrom diagrams.
compatibility: wavedrom-cli, WaveDrom Editor (wavedrom.com/editor.html)
---

# WaveDrom Timing Diagram

Generate glitch-free WaveDrom JSON timing diagrams and render them with
`wavedrom-cli`.

## Verified Resources

- Read `references/wavedrom-syntax.md` before writing any WaveJSON.
- Start from `templates/minimal.json` for new diagrams.
- Use `examples/clocked_bus.json` as the primary synchronous bus reference.
- Read `references/apb-timing.md` before drawing APB transfers. Select the
  closest single-scenario example: `examples/apb_read.json`,
  `examples/apb_read_wait.json`, `examples/apb_write.json`,
  `examples/apb_back_to_back.json`, or `examples/apb_error.json`. Use
  `examples/apb_read_write.json` only for a compact write-then-read overview.
- Use `examples/ahb_lite_read_write.json` for AHB-Lite protocol timing.
- Use `examples/spi_single_read.json`, `examples/spi_dual_read.json`, and
  `examples/spi_quad_read.json` for colored protocol phases, parallel SPI I/O,
  and compressed timelines. Read `references/spi-flash-read.md` first when
  drawing SPI NOR Flash reads.
- Render with `wavedrom-cli --input <file.json> --svg <file.svg>`.

All paths are relative to this Skill directory.

## Rendering Model

- Every `wave` character occupies one time period. A clock character toggles
  twice in that period, while `.` extends the preceding state.
- `0` and `1` request transitions into low and high. Repeating the same digit
  requests another same-level transition; WaveDrom renders this as a visible
  low-low or high-high spike rather than a flat hold.
- `=` and `2`-`9` create bus polygons. Every such character starts a distinct
  segment and consumes one label. `.` extends the current polygon without a
  crossover.
- An empty string in `data` is not missing data. It intentionally creates an
  unlabeled bus segment, keeping the lane as a double-line bus during idle.
- `phase: -0.5` shifts a non-clock lane half a period left so its value is
  stable before a rising clock edge. Use this consistently for the synchronous
  diagrams in this Skill; do not treat it as a universal WaveDrom requirement.
- `config.hscale` changes horizontal spacing only. Increase it when labels or
  adjacent one-cycle bus segments are visually crowded.
- `|` is a visual timeline gap, not an implicit value or protocol-phase
  transition. A summary lane may use `.` at that character position while
  detailed clock and data lanes use `|`.

## Required Drawing Style

- Use `wavedrom-cli`; do not consider source inspection alone sufficient.
- Include the relevant clock in synchronous bus, APB, and AHB diagrams.
- Draw protocol buses with the original continuous double-line style: blank
  bus segment, valid labeled segment, then another blank bus segment.
- Use clean, deterministic idle bus segments by default. Use hatched `x` or
  high-impedance `z` only when that state is part of the specification.
- Keep all wave strings equal length and all non-clock synchronous lanes on
  one phase convention.
- APB examples must show Setup followed by Access, hold address/control/write
  data through the completion edge, and assert `PENABLE` only in Access.
- During APB wait states, keep `PSEL`, `PENABLE`, address, direction, and write
  data stable. Complete the transfer only where `PSEL`, `PENABLE`, and
  `PREADY` are high.
- Back-to-back APB transfers must include a new Setup period with `PENABLE`
  low. Show `PSLVERR` only in the final Access period where it is meaningful.
- AHB-Lite examples must distinguish address and data phases and preserve the
  pipeline relationship between them.
- Do not accept unintended glitches, bus crossovers, clipped or overlapping
  labels, ambiguous idle shapes, or signals that are visibly misaligned with
  their sampling clock edge.
- Use the same bus color for the same semantic phase or related data group.
- Do not infer optional protocol fields such as mode bits or dummy cycles from
  a generic command name. Use the target device specification or an explicit
  user requirement.

## Key Rules

1. Keep all `wave` strings the same length as a project convention so every
   character maps to the same clock period.
2. On single-bit signals, use `0` or `1` only for an actual transition and
   `.` to hold the level. Never use `00` or `11` to hold a level: WaveDrom
   draws a same-level transition that looks like a glitch.
3. Each `=`, or colored value `2`-`9`, starts a bus data segment and consumes
   one `data` label. Use `.` to extend that segment without a crossover.
4. Consecutive `==` means two adjacent data segments. Use it only when the
   bus intentionally changes value on that cycle boundary.
5. Keep protocol buses in bus form across the full diagram: start with `=`
   paired with `""`, extend the blank segment with `.`, and return to another
   empty-labeled `=` after valid data. This avoids collapsing the bus to a
   single-bit `0` line. Use `x` or `z` only when unknown or high impedance is
   semantically required.
6. Use groups `["Group Name", {...}, {...}]` to organize related signals.
7. `phase` accepts positive or negative numbers. For rising-edge synchronous
   examples, apply `phase: -0.5` consistently to non-clock signals so they
   are visibly stable before the sampling edge.
8. `data` can be an array or a whitespace-delimited string. Prefer arrays
   when labels contain spaces or empty labels.

## Phase Summary and Timeline Compression

- Treat a `Phase` or other summary lane as a semantic overview, not as a copy
  of every detailed waveform transition.
- Start one colored bus segment and consume one label per semantic phase. Hold
  that segment with `.` until the next real phase boundary.
- Never restart the same phase with another `=` or `2`-`9` after a timeline
  gap. Doing so creates a visible crossover and a second label, making one
  phase look like two transfers.
- To compress a long interval, keep every wave string the same length, put `|`
  at the compressed character position on SCLK, CS/control, and detailed data
  lanes, and put `.` at that position on a continuous summary lane.
- Show enough cycles before and after a gap to make the omitted interval's
  boundaries unambiguous. State the omitted bit range, byte range, or clock
  count in a footnote.
- Keep colors semantic and consistent across lanes. A phase summary and its
  corresponding detailed data use the same color on both sides of a gap.

## Glitch-Free Checks

- Reject same-level repeats such as `00` and `11`; replace them with `0.` or
  `1.`. Repeated `0`/`1` is allowed only for real alternating transitions.
- Reject `==` when both positions are intended to show one stable bus value;
  replace it with `=.`.
- For protocol buses, include empty labels for blank bus segments before,
  between, and after transfers so the lane remains a double-line bus.
- Count one `data` label for every `=`, `2`, `3`, `4`, `5`, `6`, `7`, `8`, or
  `9` character.
- Keep address, control, and data stable across protocol wait states by
  extending them with `.`.
- Render every result and inspect the SVG for unintended low-low/high-high
  spikes, unwanted bus crossovers, missing labels, and clock misalignment.

## Visual Correction Loop

Do not claim visual correctness merely because JSON parsing or SVG generation
succeeded. Perform this loop for every final diagram:

1. Render SVG with `wavedrom-cli --input <file.json> --svg <file.svg>`.
2. Also render a PNG for inspection with
   `wavedrom-cli --input <file.json> --png <file.png>`.
3. Open/read the PNG and inspect the actual pixels, not only the SVG source.
4. Compare each row against the intended clock cycles and protocol phases.
5. Correct the WaveJSON and rerender until all checks pass.
6. Keep the requested SVG; remove temporary PNG inspection artifacts unless
   the user explicitly wants them.

During visual inspection, explicitly check:

- Clock edges are visible and the sampled values are stable before them.
- Bus lanes remain double-line through blank periods rather than collapsing
  into a single low line.
- Crossovers occur only at intentional value boundaries; a held value has no
  internal crossover.
- No low-low/high-high spike appears on a signal that should remain stable.
- Labels belong to the intended bus segment, are centered, and do not overlap
  neighboring labels or crossover edges.
- A summary phase appears once, remains continuous across compressed time, and
  does not gain a crossover at a gap.
- Every timeline gap is aligned across the detailed clock, control, and data
  lanes that share the omitted interval.
- APB Setup/Access and AHB address/data phases occupy the intended cycles.
- APB read data is valid at the completion edge, wait states do not alter
  request signals, and consecutive transfers include a fresh Setup period.
- Long labels remain legible; increase `hscale` and rerender when necessary.

## Workflow

1. Identify signals and the protocol/scenario to model. For APB, read
   `references/apb-timing.md` and select the nearest single-scenario example.
2. Determine the number of clock periods needed.
3. Draft each signal's `wave` string, using `.` for every held state.
4. For buses, allocate one label per data character and use `.` for holds.
5. Make wave strings equal length and apply phase consistently.
6. Run the glitch-free checks above.
7. Render SVG and a temporary PNG with `wavedrom-cli`.
8. Complete the visual correction loop; do not stop after a successful CLI
   exit code.

## Quick Reference

| Char | Meaning |
|------|---------|
| `0` / `1` | Low / High with a transition |
| `l` / `h` | Low / High without a transition |
| `.` | Extend previous state without a transition |
| `=` | Bus data segment (default color) |
| `2`-`9` | Colored bus (8 distinct colors) |
| `x` | Unknown (causes hatching on data edges) |
| `z` | High impedance |
| `p` / `n` | Positive / Negative clock edge |
| `\|` | Timeline gap separator |

## Render

```powershell
wavedrom-cli --input <file.json> --svg <file.svg>
```
