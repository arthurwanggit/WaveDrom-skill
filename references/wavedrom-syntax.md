# WaveDrom Syntax Reference

## Signal Format

```
{ "signal": [ { "name": "...", "wave": "...", "data": [...] }, ... ] }
```

`name` and `wave` are normally used for waveform lanes. Empty objects create
spacer rows, and node-only lanes are also valid. `data`, `phase`, `period`,
and `node` are optional.

---

## Wave Characters

### Logic States

| Char | Renders As |
|------|-----------|
| `0` | Low level with a transition |
| `1` | High level with a transition |
| `l` | Low level without a transition |
| `h` | High level without a transition |
| `.` | **Extend** previous state without a transition |
| `x` | Unknown - diagonal hatching, creates crossover marks on data transitions |
| `z` | High-impedance - mid-line rendering |
| `d` | Dashed low |
| `u` | Dashed high |

### Clock

| Char | Description |
|------|------------|
| `p` | Positive edge, no arrow |
| `P` | Positive edge, with arrow marker |
| `n` | Negative edge, no arrow |
| `N` | Negative edge, with arrow marker |

Each clock char toggles twice per period.

### Bus (Data)

| Char | Color |
|------|-------|
| `=` | Default (yellow, #ffffb4) |
| `2` | Yellow-orange |
| `3` | Orange |
| `4` | Blue |
| `5` | Cyan |
| `6` | Green |
| `7` | Purple |
| `8` | Pink |
| `9` | Light pink |

### Gap

| Char | Effect |
|------|--------|
| `\|` | Visual timeline break, separates distinct phases |

`|` only draws a timeline break. It does not require a protocol phase or bus
value to restart. On a semantic summary lane, use `.` at the corresponding
character position when the same phase continues across compressed time. Keep
`|` on detailed clock, control, and data lanes where the omitted interval must
remain visible.

```json
{
  "signal": [
    {"name": "Phase", "wave": "=3......4=", "data": ["", "CMD", "DATA", ""]},
    {"name": "SCLK",  "wave": "lpp|pppppl"},
    {"name": "CS#",   "wave": "10.|.....1"}
  ]
}
```

The `CMD` summary above is one continuous colored segment with one label. Do
not start another `3` after the detailed lanes' gap unless a new bus value or
phase actually begins.

---

## Bus & Data Array Rules

1. Each `=` or `2`-`9` character consumes one `data` entry.
2. Consecutive data characters create adjacent data segments and a crossover.
3. `.` after a data character extends that segment without consuming a label.
4. `data` may be an array or a whitespace-delimited string. Prefer an array
   for labels containing spaces or intentionally empty labels.
5. For a continuous protocol-bus appearance, use `=` with an empty label for
   idle, then extend it with `.`. Use `x` or `z` only when the bus is actually
   unknown or high impedance. Transitions from `x` are hatched.

```
// Stable value with blank bus segments before and after it.
{"wave": "=.=..=", "data": ["", "ADDR", ""]}

// Back-to-back values: each equals sign consumes one label.
{"wave": "===", "data": ["", "A0", "A1"]}
```

---

## Groups

```
["Group Name",
  {"name": "signal1", "wave": "..."},
  ["Nested Group",
    {"name": "signal2", "wave": "..."}
  ],
  {"name": "signal3", "wave": "..."}
]
```

Groups render with vertical brackets and rotated labels. Empty `{}` creates a blank separator row.

---

## Phase

`phase` is numeric. Negative values shift a lane left and positive values shift
it right. `"phase": -0.5` is useful for showing synchronous inputs stable for
half a cycle before a rising sampling edge, but it is not the only valid value.

`period` is a positive integer that scales the lane period.

---

## Nodes and Edges

`node` places single-character markers on a lane. Root-level `edge` entries
connect marker pairs, for example `"a->b request"` or `"a~>b response"`.
Lowercase nodes are visible; uppercase nodes can be used as hidden anchors.

---

## Structural Rules

1. Equal wave-string lengths are recommended for cycle-by-cycle protocol
   diagrams, although WaveDrom itself does not require them.
2. Write `0` or `1` only at real state changes; use `.` for a held level.
3. Never use `00` or `11` to hold a level. The same-level transition renders
   as a low-low or high-high spike.
4. Use one label per bus data character. Use `.` rather than another `=` to
   hold the same bus value.
5. Keep all synchronous non-clock lanes on the same phase convention.
6. Start protocol buses with an empty-labeled data segment so they render as
   buses rather than single-bit low lines during idle periods.
7. A summary lane consumes one data label per semantic phase, not one label per
   visible fragment around a timeline gap.

---

## Visual Validation

Successful rendering proves only that the input is accepted. Inspect a PNG of
the rendered result to catch semantic and presentation defects:

- `00` and `11` can produce visible same-level spikes.
- A stable bus written with another `=` gets an unintended crossover; use `.`.
- A bus idle written as `0` collapses to a single logic line; use an
  empty-labeled bus segment when a continuous double-line bus is intended.
- Short segments and long labels may overlap; increase `config.hscale`.
- Check protocol control and data against clock ticks after applying `phase`.

Render both formats during verification:

```powershell
wavedrom-cli --input diagram.json --svg diagram.svg
wavedrom-cli --input diagram.json --png diagram.png
```
