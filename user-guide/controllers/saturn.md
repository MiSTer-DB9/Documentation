# Sega Saturn

The Saturn path supports the standard Saturn Control Pad in 1P or 2P configurations and the 3D Control Pad with the physical switch in the **Digital** position. Saturn support is **key-gated**: it requires a valid `db9pro.key` at `/media/fat/db9pro.key`. See [../pro-key.md](../pro-key.md).

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
| `0x5` | 3D Control Pad in the **Analog** switch position | flagged "present" but no digital data extracted (use SNAC mode in the Saturn core for analog) |
| anything else | nothing connected, or floating mux input | no data committed, port reads as zero |

A 4-bit shift register debounces the detection: a pad has to be missed four scans in a row (≈572 µs) before it is treated as disconnected, but a single hit (≈143 µs) brings it back. Both ports start out marked "disconnected" — there are no ghost inputs at boot, and unconnected sides of a 2P mux adapter never leak phantom presses.

In **`1 Player`** mode with the 2P adapter, the routing automatically uses whichever side has a valid pad (P1 preferred). So plugging a single controller into the P2 socket of a 2P adapter still works.

## Saturn 3D Control Pad

The 3D Pad has a physical switch on top: **Digital** or **Analog**.

- **Digital** position: the pad reports the standard `0xB` ID and behaves like a normal Control Pad. Full button + D-pad support.
- **Analog** position: the pad reports `0x5`, which the helper detects as "Saturn pad present" but does not decode further. The 4-phase probe used here cannot retrieve the analog response — that requires the Saturn SMPC's own ID5/ANALOG handshake, which only the Saturn core itself runs.

For real analog play (twist of the joystick, analog triggers in *NiGHTS*, *Panzer Dragoon*, etc.) on the Saturn core, switch the pad to **Analog** and use the Saturn core's **SNAC mode** in its OSD (`Pad 1 SNAC`, `status[27]`). SNAC mode wires `USER_IN` and `USER_OUT` directly to the SMPC implementation in the core, bypassing the helper entirely.

On non-Saturn cores, the 3D Pad will register as "pad detected, no buttons" while the switch is in Analog. Flip the switch to Digital and you get the regular pad behaviour.

## Coin / OSD combos

| Combo | Action |
|---|---|
| `Start + B` | Coin (in arcade cores) |
| `Start + C` | Open / close OSD |
| `A` | OSD: confirm |
| `B` | OSD: back |
