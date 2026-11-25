# Custom BionX hardware

The original battery of my BionX e-bike is worn out. The new pack gets a DALY BMS with
balancing, and these boards replace what the original electronics did besides battery management.
Firmware: [BIONX_console_ESP32](https://github.com/tlacassi/BIONX_console_ESP32).

## How it works

Two ESP32-S3 boards linked by the bike's CAN bus.

- **Power Control** (near the battery): drives the motor, handles precharge, switching,
  sleep, current/voltage measurement and the link to the BMS.
- **Display** (handlebar): round screen, rotary encoder, brake and throttle inputs.

## Status

In progress.

- Power Control: routed (112 × 82 mm, 4 layers), **not manufactured yet**.
  The [audit](docs/AUDIT_PCB_POWER_CONTROL.md) (in French) lists issues to fix before ordering.
- Display: schematic only.

Designed in KiCad 10. Design choices and BOM in [`docs/`](docs/).

> Unvalidated prototype: do not manufacture or connect to a battery as is.
