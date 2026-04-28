# OSD Menu

Every MiSTer-DB9 core adds two extra OSD options for direct controllers.

## The options

| OSD entry | Values | Purpose |
|---|---|---|
| `UserIO Joystick` | `Off` / `Saturn` / `DB9MD` / `DB15` | Selects the protocol used to read the controller. `Off` reverts to USB-only behaviour. |
| `UserIO Players` | `1 Player` / `2 Players` | Whether one or two physical pads are wired (DB9MD splitter, DB15 splitter, Saturn 2P adapter with 74HC157D mux). |

The order of values is fixed: **`Off, Saturn, DB9MD, DB15`**. Saturn comes first on purpose — see the note below.

## Activating from a USB keyboard

1. Press **F12** to open the OSD.
2. Pick the core's "Settings" submenu (the location varies per core).
3. Set `UserIO Joystick` to the controller protocol you have wired.
4. Set `UserIO Players` to 1 or 2.
5. Pick **Save settings** so the choice survives a reboot.

## Activating from the DB controller itself (chicken-and-egg)

The first time you enable a DB controller, you have to use the keyboard — the controller path is still `Off`. After saving once, the OSD becomes navigable from the controller.

## Navigating the OSD with a DB controller

After `UserIO Joystick` is set and saved:

| Action | DB9 / DB15 input | Saturn input |
|---|---|---|
| Open the OSD | `Start + C` | `Start + C` |
| Move cursor | D-pad | D-pad |
| Confirm / Enter | `A` | `A` |
| Back / Exit | `B` | `B` |
| Close OSD | `Start + C` | `Start + C` |

Inserting a coin in arcade cores is the same combo as the upstream layout: `Mode` (or `Select` on DB15) alone, or `Start + B` together — whichever the core's mapping picks up. See the per-core button notes in [controllers/](controllers/).

## Why Saturn comes first

When you cycle the `UserIO Joystick` value, the OSD walks the list in order. If `DB9MD` were listed before Saturn, plugging in a Saturn pad and clicking once would activate `DB9MD` first. The DB9MD module would drive the MD `SELECT` line and read the Saturn pad's data lines as garbage button presses, sending phantom OSD inputs that make it impossible to keep clicking through to Saturn.

Putting Saturn first means: with a Saturn pad connected, the first click goes straight to the right protocol. With a Mega Drive pad connected, even though the cycle visits Saturn first, the Saturn detection logic stays inert (it requires a Saturn-specific signature that a Mega Drive pad cannot accidentally produce), so no ghost inputs reach the menu.

If you want the technical detail, see [../maintainer-guide/the Saturn OSD-cycle rule](../maintainer-guide/the Saturn OSD-cycle rule).
