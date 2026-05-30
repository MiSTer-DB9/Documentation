# Neo Geo / Supergun (DB15)

The DB15 path is built around the Antonio Villena DB15 splitter board. The splitter exposes two players' worth of buttons through a synchronous shift register clocked by the FPGA, so the protocol is independent of what the Neo Geo console itself uses on the same connector.

## OSD setting

`UserIO Joystick = DB15`, `UserIO Players = 1 Player` or `2 Players`.

## Hardware

The DB15 splitter uses three of the eight `USER_IO` lines:

| `USER_IO` line | Role |
|---|---|
| `USER_IO[0]` | `JOY_LOAD` (FPGA → splitter, push-pull) |
| `USER_IO[1]` | `JOY_CLK` (FPGA → splitter, push-pull) |
| `USER_IO[5]` | `JOY_DATA` (splitter → FPGA, input) |

The FPGA drives `LOAD` once per scan to capture the parallel button state in the splitter, then clocks `CLK` 26 times to shift the bits out on `DATA`. The full scan covers both players.

## Internal `joydb_1` bit layout

`joydb_1` is 16 bits when DB15 is selected:

| Bit | Function |
|---|---|
| 0 | Right |
| 1 | Left |
| 2 | Down |
| 3 | Up |
| 4 | A (button 1) |
| 5 | B (button 2) |
| 6 | C (button 3) |
| 7 | D (button 4) |
| 8 | E (button 5) |
| 9 | F (button 6) |
| 10 | Start |
| 11 | Select |
| 15:12 | unused |

The bottom 11 bits ([10:0]) line up byte-for-byte with the DB9MD layout — A=A, B=B, C=C, D=X, E=Y, F=Z, Start=Start. That is intentional: any core that ports for DB9MD picks up DB15 for free.

## Compatibility note

Despite the name, the DB15 path works for any controller that exposes itself through the Antonio Villena splitter — Neo Geo native pads, Supergun-style adapters, and any homebrew board that targets the same shift-register protocol. From the FPGA's point of view, only the splitter exists; the controller behind it is opaque.

## Coin / OSD combos

| Combo | Action |
|---|---|
| `Select` (button 11) | Coin |
| `Start + C` | Open / close OSD |
| `A` | OSD: confirm |
| `B` | OSD: back |

`Select` is the bit-11 button — the same `joydb_1[11]` slot that DB9MD exposes as `Mode`. Arcade cores wire that slot to Coin, so `Select` inserts a coin on cores that expose a coin input; cores that don't simply ignore it. The DB15 splitter has a real `Select`, so there is no `Start + B` chord here — that `Start + B → Mode` synthesis exists only on the DB9MD path for 3-button MD pads (which lack a `Mode` button). `Start + C` opens / closes the OSD on every core.
