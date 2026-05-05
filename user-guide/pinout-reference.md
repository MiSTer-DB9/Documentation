# Pinout Reference

This page documents the wiring between the DE10-Nano `USER_IO` connector (Arduino-style header on the GPIO breakout) and the DB9 / DB15 / Saturn / SNAC8 connectors used by MiSTer-DB9.

The DE10-Nano `USER_IO` is the same physical port as the USB3-style connector used by some adapter boards. The table below shows the equivalence.

## USER_IO ↔ DE10-Nano ↔ DB9 ↔ FPGA ↔ USB3

| USER_IO | DE10-Nano (Arduino label) | DB9 pin | FPGA pin | USB3 pin |
|---|---|---|---|---|
|  | — | 8 | — | 4 (GND) |
|  | — | 5 | — | 1 (5V) |
| `USER_IO[0]` | `Arduino_IO15` | 7 | `AG11` | 3 (D+) |
| `USER_IO[1]` | `Arduino_IO14` | 3 | `AH9` | 2 (D−) |
| `USER_IO[2]` | `Arduino_IO13` | 4 | `AH12` | 8 (TX−) |
| `USER_IO[3]` | `Arduino_IO12` | 6 | `AH11` | 7 (GND_D) |
| `USER_IO[4]` | `Arduino_IO11` | shield | `AG16` | 6 (RX+) |
| `USER_IO[5]` | `Arduino_IO10` | 1 | `AF15` | 5 (RX−) |
| `USER_IO[6]` | `Arduino_IO8`  | 9 | `AF17` | 9 (TX+) |
| `USER_IO[7]` *(MiSTer-DB9 extension)* | `Arduino_IO9` | 2 | `AE15` | — |

`USER_IO[7]` is the bit MiSTer-DB9 adds on top of upstream's 7-bit user port: it brings out DB9 pin 2, which the DB9MD protocol needs for the Down direction. The pin had been used for `SD_SPI_CS` (secondary SD); MiSTer-DB9 reassigns it. Cores that need a secondary SD card cannot coexist with this build on the same hardware.

> **Voltage note.** On the upstream "regular SNAC" wiring, the `TX+` line on the USB3-style connector is tied to 3.3 V. On the "SNAC+" variant it is `AF17` (`USER_IO[6]`). If you build or buy an adapter, make sure it matches the variant you intend to drive — feeding 3.3 V into an FPGA pin that expects a logic-level signal can damage the DE10-Nano.

## DB9 → controller-protocol pin role per `UserIO Joystick` mode

The same physical wires carry different signal roles depending on which protocol is selected. The push-pull / open-drain status of each pin also changes — that is what the MiSTer-DB9 fork's `USER_PP` mask configures internally.

| Pin (DB9) | DB9MD (Mega Drive) | DB15 (Villena) | Saturn |
|---|---|---|---|
| 1 (`USER_IO[5]`) | Up | `JOY_DATA` | `D2` |
| 2 (`USER_IO[7]`) | Down | (N/C) | (N/C) |
| 3 (`USER_IO[1]`) | Left | `JOY_CLK` (push-pull) | `D0` |
| 4 (`USER_IO[2]`) | Right | (N/C) | 2P split-select (push-pull) |
| 5 | +5 V | +5 V | +5 V |
| 6 (`USER_IO[3]`) | B / A button | (N/C) | `D3` |
| 7 (`USER_IO[0]`) | `MD_SELECT` / `TH` (push-pull) | `JOY_LOAD` (push-pull) | `D1` |
| 8 | GND | GND | GND |
| 9 (`USER_IO[6]`) | C / Start | (N/C) | `S1` (push-pull) |
| shield (`USER_IO[4]`) | (shield/GND) | (N/C) | `S0` (push-pull) |

The Saturn 2P mux adapter additionally uses `USER_IO[2]` to drive the 74HC157D split-select: low = port A, high = port B.

## SNAC8 (eight-pin direct controller)

SNAC8 is a per-core, lower-level form: the core takes raw access to the eight `USER_IO[7:0]` pins and drives whatever protocol the original console used (NES, SNES, PC Engine, etc.). There is no fork-level pinout here — each core defines its own. See [controllers/snac8.md](controllers/snac8.md) for the list of cores that implement it.
