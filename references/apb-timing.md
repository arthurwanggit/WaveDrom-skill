# APB Timing Reference

Use this reference for AMBA APB transfer timing. It focuses on the protocol
rules needed to draw an accurate waveform rather than reproducing the full
AMBA specification.

## Version Scope

- APB2 provides the basic address, select, enable, direction, and data signals.
- APB3 adds `PREADY` wait-state support and `PSLVERR` error reporting.
- APB4 adds signals such as `PSTRB` and `PPROT`.
- Include version-specific signals only when the requested interface uses
  them. Do not imply that `PREADY`, `PSLVERR`, `PSTRB`, or `PPROT` exist in
  every APB version.

## Transfer Phases

Every transfer has at least two clock periods:

1. **Setup**: assert `PSEL`, keep `PENABLE` low, and present address, direction,
   and write data or optional control information.
2. **Access**: keep `PSEL` asserted, assert `PENABLE`, and hold the transfer
   information stable.
3. **Complete**: the transfer completes on the rising edge where `PSEL`,
   `PENABLE`, and `PREADY` are all high. For an interface without `PREADY`, the
   transfer completes at the end of the first Access period.

`PENABLE` must never be high during Setup. It remains high through all Access
wait states and returns low after the transfer completes.

## Signal Stability

Hold these signals stable from Setup through the completion edge:

- `PADDR`
- `PWRITE`
- `PWDATA` for writes
- `PSTRB` and `PPROT` when present
- the active `PSEL` for the selected peripheral

When `PREADY` is low during Access, extend the existing values. Do not create
new bus segments or crossovers for unchanged address, control, or write data.

For reads, the slave drives `PRDATA`. The master samples it on the completion
edge. A drawing may show `PRDATA` becoming valid earlier, but it must be valid
for the final Access period where `PREADY` is high.

## Back-to-Back Transfers

- Every new transfer starts with a new Setup period, so `PENABLE` must go low
  for one period between Access periods.
- `PSEL` may remain high for consecutive transfers to the same peripheral.
- Address, direction, and data may change at the boundary into the new Setup
  period, never in the middle of an Access period.
- When the next transfer selects a different peripheral, deassert the old
  peripheral's select and assert the new select during the new Setup period.

## Error Response

For APB3 and later, `PSLVERR` is only meaningful in the final Access period
where `PSEL`, `PENABLE`, and `PREADY` are high. Keep it low or visually
unasserted at other times. An error response still completes the APB transfer.

## Drawing Conventions

- Include `PCLK` and use one character per APB clock period.
- Use `phase: -0.5` consistently on non-clock lanes in the provided examples.
- Keep address and data lanes in continuous bus form, including empty-labeled
  bus segments before and after the transfer.
- Use `.` to hold every stable value, especially through wait states.
- A phase-summary lane may distinguish Setup and Access visually, but a long
  Access phase with wait states remains one continuous semantic segment.
- Label a completion condition in the footer when waits or errors are central
  to the scenario.

## Example Selection

| Scenario | Example |
|----------|---------|
| Basic read, no wait state | `examples/apb_read.json` |
| Read with inserted wait states | `examples/apb_read_wait.json` |
| Basic write, no wait state | `examples/apb_write.json` |
| Consecutive transfers | `examples/apb_back_to_back.json` |
| Error response | `examples/apb_error.json` |
| Compact write-then-read overview | `examples/apb_read_write.json` |

## Common Errors

- Asserting `PENABLE` in the Setup period.
- Dropping `PSEL` while `PREADY` is low.
- Changing `PADDR`, `PWRITE`, or `PWDATA` during an Access wait state.
- Treating each wait period as a new transfer or a new bus value.
- Sampling `PRDATA` before the completion edge.
- Holding `PSLVERR` high outside the final Access period.
- Drawing consecutive Access periods without the intervening Setup period.
