# Supported Controllers — Overview

| Controller | OSD setting | Connector | Adapter form | Pages |
|---|---|---|---|---|
| Sega Mega Drive 3-button & 6-button (a.k.a. Genesis) | `DB9MD` | DB9 | Direct, or 1P/2P Antonio Villena splitter | [mega-drive.md](mega-drive.md) |
| Sega Master System 2-button | `DB9MD` (auto-detect) | DB9 | Same as Mega Drive | [mega-drive.md](mega-drive.md) |
| Atari 2600 / 7800 stick (1-button) | `DB9MD` (auto-detect) | DB9 | Same as Mega Drive | [mega-drive.md](mega-drive.md) |
| Neo Geo / Supergun (Antonio Villena DB15) | `DB15` | DB15 | Antonio Villena DB15 splitter | [neogeo-supergun.md](neogeo-supergun.md) |
| Sega Saturn digital pad (Control Pad) | `Saturn` | Saturn-style mini-DIN or DB9 wired equivalent | 1P direct, or 2P with 74HC157D mux | [saturn.md](saturn.md) |
| Sega Saturn 3D Control Pad | Saturn core SNAC mode only (not UserIO) | Saturn-style | 1P passive cable + `Saturn SNAC Adapter = 1P` | [saturn.md](saturn.md) |
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

The DB9MD, DB15, and Saturn-digital paths are all **digital only**. The Saturn 3D Control Pad uses the Saturn 3-wire `TH`/`TR`/`TL` handshake (header `0x02` in Digital, `0x16` in Analog) in **both** switch positions, so the UserIO=Saturn 4-phase helper cannot read it in either mode — it appears as flag-present but commits no button data. For the 3D Pad, use the Saturn core's `Saturn SNAC = ON` with `Saturn SNAC Adapter = 1P`; the core's SMPC implementation talks to the pad directly and supports both Digital and Analog switch positions.
