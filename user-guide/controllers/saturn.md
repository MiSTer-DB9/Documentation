# Sega Saturn

The Saturn UserIO path supports the standard Saturn Control Pad in 1P or 2P configurations. The **3D Control Pad is NOT supported via UserIO** in either switch position — it uses a different protocol (Saturn 3-wire handshake on TH/TR/TL); use the Saturn core's SNAC mode for the 3D Pad. Saturn support is **key-gated**: it requires a valid `db9pro.key` at `/media/fat/db9pro.key`. See [../pro-key.md](../pro-key.md).

## OSD setting

`UserIO Joystick = Saturn`, `UserIO Players = 1 Player` or `2 Players`.

## Hardware

Saturn uses four signal lines (D0..D3) plus two select lines (S0, S1) plus, for the 2P mux adapter, a split-select line:

| `USER_IO` line | Role | Direction |
|---|---|---|
| `USER_IO[1]` | `D0` | input |
| `USER_IO[0]` | `D1` | input |
| `USER_IO[5]` | `D2` | input |
| `USER_IO[3]` | `D3` | input |
| `USER_IO[4]` | `S0` (push-pull) | output |
| `USER_IO[6]` | `S1` (push-pull) | output |
| `USER_IO[2]` | 2P mux split-select (push-pull) | output (Saturn 2P only) |

The FPGA scans the controller through the four phases `{S0,S1} = {0,0}, {1,0}, {0,1}, {1,1}` and assembles a 16-bit value per port.

For the 2P adapter, the split-select on `USER_IO[2]` toggles between port A (low) and port B (high) on the 74HC157D mux board.

## Internal `joydb_1` bit layout

`joydb_1` is 16 bits when Saturn is selected:

| Bit | Function |
|---|---|
| 0 | Right |
| 1 | Left |
| 2 | Down |
| 3 | Up |
| 4 | A |
| 5 | B |
| 6 | C |
| 7 | X |
| 8 | Y |
| 9 | Z |
| 10 | Start |
| 11 | unused |
| 12 | L trigger |
| 13 | R trigger |
| 15:14 | unused |

Bits [10:0] line up with DB9MD and DB15. Bits [13:12] (the shoulder triggers) are Saturn-only.

## Pad detection (the reason 2P adapters don't ghost-input)

The helper module computes a 4-bit `MD_ID` from the pad's response at the `{S0=1, S1=1}` phase and only commits button data when the ID matches a known pad family:

| `MD_ID` | Pad type | Behaviour |
|---|---|---|
| `0xB` | Standard 6-button digital Control Pad | full button data committed |
| `0x5` | 3D Control Pad in the **Analog** switch position | flagged "present" only (helper accidentally OR-reduces the 3-wire response to `0x5`); no digital data extractable here — use the Saturn core's SNAC mode |
| `0xA` | Saturn Stunner / Virtua Gun (HSS-0152) | flagged "present" only — use the Saturn core's SNAC mode (SMPC auto-detects Stunner via `PS_NOTHING_STUNNER`); set `Saturn SNAC Adapter = 1P` if you are on the passive cable |
| anything else | nothing connected, floating mux input, or 3D Pad in **Digital** switch position (uses 3-wire handshake, not the 4-phase protocol) | no data committed, port reads as zero |

A 4-bit shift register debounces the detection: a pad has to be missed four scans in a row (≈572 µs) before it is treated as disconnected, but a single hit (≈143 µs) brings it back. Both ports start out marked "disconnected" — there are no ghost inputs at boot, and unconnected sides of a 2P mux adapter never leak phantom presses.

In **`1 Player`** mode with the 2P adapter, the routing automatically uses whichever side has a valid pad (P1 preferred). So plugging a single controller into the P2 socket of a 2P adapter still works.

## Saturn 3D Control Pad

The 3D Pad has a physical switch on top: **Digital** or **Analog**. It uses the Saturn 3-wire handshake protocol (TH/TR/TL) in **both** switch positions (header byte `0x02` in Digital, `0x16` in Analog), unlike the standard 6-button digital Control Pad which uses a 4-phase TH/TR scan with no TL handshake. This means:

- **UserIO=Saturn does not support the 3D Pad in either switch position.** The `joydb9saturn` helper only implements the 4-phase scan; the 3-wire handshake state machine is not present. The pad will appear silent on D-lines.
- **The 3D Pad works only via the Saturn core's SNAC mode**, where the Saturn core's own SMPC implementation runs the full 3-wire handshake.

### Using the 3D Pad on a Saturn 1P passive SNAC adapter

1. Open the Saturn core's OSD.
2. Set `Saturn SNAC = ON`.
3. Set `Saturn SNAC Adapter = 1P` (the default is `2P` — for the 74HC157D-mux modded adapter).
4. Plug the 3D Pad into the SNAC port. Either switch position works (Digital for *Daytona*-style games, Analog for *NiGHTS*, *Panzer Dragoon*, *Sega Rally*, etc.).

`Saturn SNAC Adapter = 1P` releases every SNAC pin to open-drain, including `USER_IO[2]`, so the FPGA does not short the pad's open-collector `TL` line. SMPC then drives `TL` via `PDR_O[4]` through the same path. The default `Saturn SNAC Adapter = 2P` drives `USER_IO[2]` push-pull as the mux SEL (correct for 6-button pads on the 2P-modded adapter), which is incompatible with the 3D Pad on a 1P passive cable.

`Saturn SNAC Adapter` is a Saturn-SNAC sub-option and inherits the same key gate as the rest of Saturn — no separate `feature_mask` bit. SMPC handles all pad-type detection (Saturn Pad, 3D Pad, Stunner, Mouse, Wheel, Mission, Dual Mission) automatically on either adapter; you only flip the selector to match your cable.

Note: with `Saturn SNAC Adapter = 1P`, the `SNAC Players` selector is internally force-overridden to single-player (the 1P passive cable has only one physical jack). Setting it to `2 Players` has no effect — and that override is what stops the 2-Players scan from injecting phantom P2 inputs from the same single port.

### 2P SNAC adapter + 3D Pad

The 2P 74HC157D mux only handles the 4 D-lines and has no spare channel for TL routing. **Stock 2P adapter cannot read the 3D Pad.** A future hardware mod (single jumper from P1 jack pin 4 to DB9 pin 2 → `USER_IO[7]`, analogous to the existing Stunner 2P mod) plus a firmware option could enable P1-only 3D Pad support; not implemented yet.

## Coin / OSD combos

| Combo | Action |
|---|---|
| `Start + B` | Coin (in arcade cores) |
| `Start + C` | Open / close OSD |
| `A` | OSD: confirm |
| `B` | OSD: back |
