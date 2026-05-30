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
| A | A |
| B | B |
| X | L |
| Y | R |
| Start | Start |
| Mode | Select |

### Game Boy Advance (option 2)

| Mega Drive | GBA |
|---|---|
| A | B |
| B | A |
| X | L |
| Y | R |
| Start | Start |
| Mode | Select |

The two options (selected by the core's *Buttons Mapping* OSD setting) only swap `A`/`B`. `C` and `Z` are unused; there is no Fast-Forward / Rewind on the DB9 pad.

### Neo Geo

| Mega Drive | Neo Geo |
|---|---|
| A | A |
| B | B |
| C | C |
| X | D |
| Y | Select |
| Z | A+B+C |
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

### Atari 7800 (also runs 2600 titles)

| Mega Drive | Atari 7800 |
|---|---|
| A | Fire 1 |
| B | Fire 1 |
| C | Fire 2 |
| Start | Reset (Start) |
| Mode | Select |

(`A` and `B` both map to Fire 1; `C` is Fire 2. There is no separate `Atari2600_MiSTer` core — the 7800 core plays 2600 titles.)

### Game Boy / Game Boy Color

Default (option 1):

| Mega Drive | GB / GBC |
|---|---|
| A | A |
| B | B |
| Start | Start |
| Mode | Select |
| Start + B | Select |

Option 2 swaps `A`/`B` (A→B, B→A); everything else is identical. `C`, `X`, `Y`, `Z` are unused. `Start + B` is wired to **Select** on this core (it is *not* a coin combo).

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
| Start + B (3-button pad only) | Coin |
| Start + C (combo) | Open / close OSD |

Coin comes from the `joydb_1[11]` (`Mode`) slot. A **6-button** MD pad has a real `Mode` button. A **3-button** MD pad has none, so the DB9MD helper (`joydb9md.v`) **synthesizes `Mode` from a held `Start + B` chord** (~105 ms debounce; `Start` and `B` are suppressed from the game while it fires, so you don't also get Pause+B). The chord is gated to *3-button MD pads only* (`~6btn`) — on a 6-button pad you press `Mode` directly and the chord is disabled. `Start + C` opens / closes the OSD on every core (`joydb_1[10] & joydb_1[6]` in the shared mux).

> The `Start + B → Mode` synthesis is **DB9MD-3-button-only**. A **DB15** pad carries its own `Select` on bit 11; a **Saturn** pad has no `Mode` bit (bit 11 is the `R` trigger). So Coin is `Start + B` (3-button MD), `Mode` (6-button MD), `Select` (DB15), or `R` (Saturn). On cores that map the bit-11 slot to a gameplay function instead of Coin (e.g. `Select` on NeoGeo, SNES, GBA, Game Boy), that same input produces *that* function.
