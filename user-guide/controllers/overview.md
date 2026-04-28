# Supported Controllers — Overview

| Controller | OSD setting | Connector | Adapter form | Pages |
|---|---|---|---|---|
| Sega Mega Drive 3-button & 6-button (a.k.a. Genesis) | `DB9MD` | DB9 | Direct, or 1P/2P Antonio Villena splitter | [mega-drive.md](mega-drive.md) |
| Sega Master System 2-button | `DB9MD` (auto-detect) | DB9 | Same as Mega Drive | [mega-drive.md](mega-drive.md) |
| Atari 2600 / 7800 stick (1-button) | `DB9MD` (auto-detect) | DB9 | Same as Mega Drive | [mega-drive.md](mega-drive.md) |
| Neo Geo / Supergun (Antonio Villena DB15) | `DB15` | DB15 | Antonio Villena DB15 splitter | [neogeo-supergun.md](neogeo-supergun.md) |
| Sega Saturn digital pad (Control Pad) | `Saturn` | Saturn-style mini-DIN or DB9 wired equivalent | 1P direct, or 2P with 74HC157D mux | [saturn.md](saturn.md) |
| Sega Saturn 3D Control Pad | `Saturn` (digital position only) | Saturn-style | Same as Saturn digital | [saturn.md](saturn.md) |
| Original-console pad through SNAC8 (NES, SNES, PCE, etc.) | per-core SNAC option | varies per console | Per-console SNAC8 adapter | [snac8.md](snac8.md) |

## Choosing the right `UserIO Joystick` value

| You have... | Set `UserIO Joystick` to |
|---|---|
| One Mega Drive controller plugged into a single-port DB9 adapter | `DB9MD`, `1 Player` |
| Two Mega Drive controllers via the Antonio Villena DB9 splitter | `DB9MD`, `2 Players` |
| One Master System / Atari pad on a DB9 adapter | `DB9MD`, `1 Player` (DB9MD auto-detects 2-button or 1-button pads) |
| Neo Geo or Supergun with the DB15 splitter | `DB15`, `1 Player` or `2 Players` |
| One Saturn Control Pad on a 1P direct adapter | `Saturn`, `1 Player` |
| Two Saturn Control Pads via the 74HC157D 2P mux adapter | `Saturn`, `2 Players` |
| One Saturn Control Pad plugged into the 2P-mux adapter (P1 *or* P2) | `Saturn`, `1 Player` (the auto-route picks whichever side has a pad) |

## DB9MD vs DB15 vs Saturn — quick guide

- **DB9MD** is auto-detecting: it figures out at runtime whether the pad is 3-button (Master System / Atari era) or 6-button (Mega Drive Mega Pad). You don't need to flip a switch on the controller.
- **DB15** uses a shift-register clocked by the FPGA. The Antonio Villena DB15 splitter has its own internal protocol that goes far beyond Neo Geo: any controller wired to its bit layout works.
- **Saturn** uses the original Saturn 4-phase select protocol (S0/S1 select lines), with per-port presence detection so an empty side of a 2P mux adapter does not produce phantom inputs.

## What about analog?

The DB9MD, DB15, and Saturn-digital paths are all **digital only**. Analog stick / trigger handling on the Saturn 3D Control Pad, in particular, requires a different protocol that the simple 4-phase probe cannot drive. For real analog play with a 3D Pad, the Saturn core itself has a "SNAC mode" (`Pad 1 SNAC` in the Saturn core OSD) that bypasses MiSTer-DB9's helper module entirely and lets the core's SMPC implementation talk directly to the pad.
