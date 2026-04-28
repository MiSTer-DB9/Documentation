# Sega Mega Drive / Genesis (DB9MD)

The DB9MD path supports any controller that uses the Sega DB9 protocol family: the original Mega Drive 3-button pad, the Mega Drive 6-button (Mega Pad / Mega Pad 2), the Sega Master System 2-button, and Atari-style 1-button sticks. Auto-detection runs at boot — you do not pick "3-button" vs "6-button" in the OSD.

## OSD setting

`UserIO Joystick = DB9MD`, `UserIO Players = 1 Player` or `2 Players` depending on whether you have the 2P splitter.

## Hardware

The signal pins on the DB9 connector follow the standard Sega layout. See [../pinout-reference.md](../pinout-reference.md) for the FPGA-side mapping.

| DB9 pin | Mega Drive role |
|---|---|
| 1 | Up |
| 2 | Down |
| 3 | Left (or `0` after `SELECT` toggles low) |
| 4 | Right (or `0` after `SELECT` toggles low) |
| 5 | +5 V |
| 6 | C / A button |
| 7 | `SELECT` / `TH` (FPGA → controller) |
| 8 | GND |
| 9 | Start / B button |

The 6-button extension uses two extra `SELECT` pulses to expose `X / Y / Z / Mode`.

## Internal `joydb_1` bit layout

When `UserIO Joystick = DB9MD`, the FPGA presents button data to each core through the 12-bit `joydb_1` (and `joydb_2` for player 2):

| Bit | Function |
|---|---|
| 0 | Right |
| 1 | Left |
| 2 | Down |
| 3 | Up |
| 4 | A (button 1) |
| 5 | B (button 2) |
| 6 | C (button 3) |
| 7 | X (button 4 — 6-button pad only) |
| 8 | Y (button 5 — 6-button pad only) |
| 9 | Z (button 6 — 6-button pad only) |
| 10 | Start |
| 11 | Mode |

How each core maps these to its in-game buttons is per-core. For cores that do not have a Mega Drive pad as their native controller, the project ships a default mapping based on the Genesis pad — see the tables below.

## Mapping to other systems (Mega Drive pad as the input device)

These tables show how MiSTer-DB9 cores route a Mega Drive pad into a non-Mega-Drive core when no specific override is configured. Some systems offer two layouts; pick the one that matches your muscle memory.

### Super Nintendo (option 1)

| Mega Drive | SNES |
|---|---|
| A | Y |
| B | B |
| C | A |
| X | L |
| Y | X |
| Z | R |
| Start | Start |
| Mode | Select |

### Super Nintendo (option 2)

| Mega Drive | SNES |
|---|---|
| A | B |
| B | A |
| C | R |
| X | Y |
| Y | X |
| Z | L |
| Start | Start |
| Mode | Select |

### Game Boy Advance (option 1)

| Mega Drive | GBA |
|---|---|
| A | B |
| B | A |
| C | R |
| X | Fast Forward |
| Y | L |
| Z | Rewind |
| Start | Start |
| Mode | Select |

### Game Boy Advance (option 2)

| Mega Drive | GBA |
|---|---|
| A | B |
| B | A |
| C | R |
| X | Fast Forward |
| Y | Rewind |
| Z | L |
| Start | Start |
| Mode | Select |

### Neo Geo

| Mega Drive | Neo Geo |
|---|---|
| A | A |
| B | B |
| C | C |
| X | Select |
| Y | A+B+C |
| Z | D |
| Start | Start |
| Mode | Coin |

### NES (option 1)

| Mega Drive | NES |
|---|---|
| B | B |
| C | A |
| Start | Start |
| Mode | Select |

### NES (option 2)

| Mega Drive | NES |
|---|---|
| B | A |
| C | B |
| Start | Start |
| Mode | Select |

### Master System

| Mega Drive | Master System |
|---|---|
| B | 1 |
| C | 2 |
| Start | Start / Pause |

### TurboGrafx-16 / PC Engine

| Mega Drive | TurboGrafx |
|---|---|
| A | III |
| B | II |
| C | I |
| X | IV |
| Y | V |
| Z | VI |
| Start | Run |
| Mode | Select |

### Atari 2600

| Mega Drive | Atari 2600 |
|---|---|
| B | Fire |
| Start | Start |
| Mode | Select |

### Game Boy / Game Boy Color

| Mega Drive | GB / GBC |
|---|---|
| B | A |
| C | B |
| Start | Start |
| A | Select |
| Mode | Select |

### Generic arcade

| Mega Drive | Arcade |
|---|---|
| A | Button 1 (Fire) |
| B | Button 2 (Jump) |
| C | Button 3 |
| X | Button 4 |
| Y | Button 5 |
| Z | Button 6 |
| Start | Start |
| Mode | Coin |
| Start + B (combo) | Coin |
| Start + C (combo) | Open OSD |

The `Start + B = Coin` combo is built into the mux logic for arcade cores: any time pin Start and pin B are pressed simultaneously, the same coin signal is asserted. This gives you a coin without a dedicated `Mode` button on a 3-button pad. The `Start + C = OSD` combo opens / closes the OSD on every core.
