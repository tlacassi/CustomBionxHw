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
