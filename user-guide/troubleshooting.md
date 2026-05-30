# Troubleshooting

If the symptom you see is in this list, the cause is usually well-known. Each entry has a quick diagnosis and a one-line fix.

## OSD cursor jumps when I switch `UserIO Joystick` away from `Off`

**You are doing**: cycling `UserIO Joystick` to enable a Saturn pad.

**You see**: cursor moves on its own, random items get selected, OSD opens and closes by itself.

**Cause**: Either the Saturn pad is producing ghost button presses through a transient mode, or the cycle visited the wrong protocol. With the correct OSD ordering (Saturn first, then DB9MD, then DB15) this should not happen. If it does:

- You are using a stock upstream `MiSTer` or `menu.rbf`, not the MiSTer-DB9 build. See [installation.md](installation.md).
- Your Saturn cable has a wiring fault that drives the data lines while a DB9MD-mode probe runs. Check the cable.
- The core is an old build that pre-dates the Saturn-first ordering fix. Update the core.

## Saturn option in OSD does nothing

**You see**: `UserIO Joystick` cycles through `Off / Saturn / DB9MD / DB15` but `Saturn` does not change controller behaviour.

**Cause**: The key gate is locked. Saturn is key-gated, so without a valid `db9pro.key` the option is decorative. See [pro-key.md](pro-key.md).

## Phantom button presses with the 2P Saturn adapter when only one pad is plugged in

**You see**: With the 74HC157D mux adapter and only one controller, the empty port shows random button presses or directionals.

**Cause**: The unconnected mux inputs float, and CMOS pins without external pulls can settle either way.

**Fix**: This is what the per-port detection in `joydb9saturn.v` is meant to suppress. With both ports requiring a valid Saturn signature (`MD_ID == 0xB` for digital, `0x5` for the 3D Pad in analog) plus a 4-bit shift-register debounce before any data is committed, you should never see this. If you do:

- The core is a pre-fix build. Update it.
- One side is wired but the cable is intermittent — the detection is engaging and disengaging fast enough to leak a sample. Reseat the connector.

If you want the technical detail, see [the Saturn 2P ghost-input notes](https://github.com/MiSTer-DB9/Documentation-Maintainer/blob/main/maintainer-guide/the Saturn-2P-ghost rule).

## OSD becomes very laggy on cores that have both DB9 and MT32-pi

**You see**: OSD redraws stutter or hang for seconds at a time. SD activity may be involved.

**Cause**: Both `MT32-pi` and the DB9 controller share the same `USER_IO` pins. With both wired and the MT32 module not gated correctly, the MT32 I2C slave sees DB9 button states as if they were I2C traffic. Random patterns can match an I2C address (`0x3C` / `0x45`), latch the MT32-available state on permanently, and trigger a cascade of OSD repaints that stalls on `IO_WAIT` during floppy/DMA activity.

**Fix**:

- Make sure the core is a recent MiSTer-DB9 build with both anti-contention gates applied (Minimig-AGA, Atari ST, X68000, ao486, TRS-80).
- If both features must coexist on the same hardware, use a **USERIO2** build of the core where available. USERIO2 routes MT32 to a separate set of FPGA pins, so DB9 and MT32 don't share the bus. Look for `<core>_USERIO2.rbf` in the core's release folder.
- If you don't actually use MT32-pi, set the OSD's MT32 option to `Disable`. That short-circuits the contention regardless.

If you want the technical detail, see [the MT32-pi contention notes](https://github.com/MiSTer-DB9/Documentation-Maintainer/blob/main/maintainer-guide/the MT32 anti-contention rule).

## DB9MD pad works for OSD navigation but in-game buttons do nothing

**Cause**: The OSD path and the in-game path are independent. While the OSD is open the core *live-detects* the pad and routes it for navigation no matter what `UserIO Joystick` is set to; in-game it uses the *saved* `UserIO Joystick` value. So if that value is on the wrong protocol — e.g. `DB15` while a Mega Drive pad is plugged in — the pad navigates the OSD fine but is misread the moment you close it.

**Fix**:

- With `USERIO_AUTO_SELECT=1` (see [osd-menu.md](osd-menu.md)), just press a button on the pad while the OSD is open — the setting snaps to the detected protocol and the pad then works in-game too. Recent fork builds do this automatically.
- Otherwise set `UserIO Joystick` to the protocol your pad actually uses from the keyboard, then **Save settings**.
- If the value already matches the pad and you only see this on **one** specific core, that core's in-game mapping was probably mis-ported — open an issue on the core's repository.

## Pressing buttons during OSD navigation leaks into the game

**You see**: Inserting coins or firing weapons happens "in the background" while you are in the menu.

**Cause**: The DB controller mux is missing the `OSD_STATUS` gate that zeroes input while the OSD is open. This is a port bug, not a configuration issue. Update the core; if the latest build still does it, file an issue against that core.

## After unplugging the SD card, the Pro key gate stays unlocked next boot

**Cause**: Wishful thinking. The gate is negotiated fresh on every core load — load any core and the gate locks immediately if no valid key is present. There is no persistent unlock.

## I want to know why a feature requires `db9pro.key`

See [pro-key.md](pro-key.md). Short version: feature licensing is customer-supported; the gated feature (Saturn) requires a small per-customer key file at `/media/fat/db9pro.key`. The file expires periodically.
