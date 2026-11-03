# BionX Hardware — KiCad starter

KiCad 10 project skeleton with a system-level hierarchical schematic and two independent board projects:

- `bionxHW.kicad_pro` / `bionxHW.kicad_sch`: top-level architecture, with sheets for the display and power-control boards.
- `pcb_display/BionX_Display.kicad_pro`: independent display-board project, schematic and blank PCB.
- `pcb_power_control/BionX_Power_Control.kicad_pro`: independent power-control-board project, schematic and blank PCB.

The two PCB project folders are intentionally independent so each schematic produces its own board. The master sheet is an architecture overview, not a combined board netlist.

## Known project context

The existing firmware target is HUZZAH32 / ESP32 with LCD and MCP2515 CAN. Treat these as context only: the exact MCU/module, display, CAN transceiver, power path, connector, supply rails and protection parts for these new boards have not been established here.

## Interface and safety constraints

The display-to-power-control connector and its pinout are TBD. No voltage, current, CAN termination, isolation, fuse, reverse-polarity, transient or thermal design is asserted by this starter. Confirm the actual BionX bus and requirements before placing parts or assigning electrical nets. Do not treat these blank PCB files as manufacturable.

## First component selection (schematic draft)

- Display board: Espressif `ESP32-S3-WROOM-1-N8` (JLCPCB `C2913198`, extended listing) and TI `SN65HVD230QDR` (JLCPCB `C16468`, extended listing). The ESP32-S3 provides a TWAI controller compatible with Classical CAN 2.0; it still needs an external physical-layer transceiver.
- Power-control board: TI `LM5069MM-1/NOPB` (JLCPCB `C3177598`, extended listing) for controlled turn-on/inrush and latched-off fault handling. Its external power MOSFET, sense/timer values and thermal implementation are not selected because controller input capacitance and startup requirements are unknown.
- JLC stock and classification change; recheck the exact LCSC/JLCPCB part numbers before ordering.

These are placed starter components, not a wired or validated design. No power path is approved for a 30 A pack yet.
