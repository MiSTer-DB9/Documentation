# User Guide

This section covers everything an end user needs to play games with original-hardware controllers on MiSTer-DB9.

## Pages

- **[installation.md](installation.md)** — Which files to download, where to put them on the SD card, and the boot sequence to expect.
- **[osd-menu.md](osd-menu.md)** — Activating the "UserIO Joystick" option, navigating the OSD with the controller (Start+C / A / B), and player-count selection.
- **[define-db9-buttons.md](define-db9-buttons.md)** — Remapping which physical button does what, saved per core/per controller type (the DB9 equivalent of USB "Define buttons").
- **[pro-key.md](pro-key.md)** — What `db9pro.key` is, where it lives, what it unlocks, and how to renew an expired key.
- **[pinout-reference.md](pinout-reference.md)** — Complete pinout for the DE10-Nano `USER_IO` connector mapped to DB9 / SNAC8 / FPGA pins.
- **[troubleshooting.md](troubleshooting.md)** — Symptoms of the known hardware/software interactions and how to recognise them.
- **[controllers/](controllers/)** — One page per supported controller family. Pinout, button order, special notes, and Genesis-pad button mappings for cores that don't have a native pad.

## What you need to get started

1. A DE10-Nano running MiSTer FPGA.
2. A wiring adapter that brings out the `USER_IO` pins to a DB9 (Mega Drive style), DB15 (Antonio Villena splitter style), or Saturn-style connector. Adapter design is hardware — not covered in this repo.
3. The MiSTer-DB9 `MiSTer` binary and `menu.rbf` (replace the upstream ones — see [installation.md](installation.md)).
4. For Saturn support only: a valid `db9pro.key` file at `/media/fat/db9pro.key`. DB9MD, DB15, SNAC8, and the MT32-pi anti-contention gate work without a key.

## What this fork does NOT change

- USB controllers continue to work exactly as on stock MiSTer.
- Cores keep their stock USB-mapped joystick code; the DB controller is muxed in only when the user explicitly selects it in the OSD.
- No firmware on the controller side. Mega Drive / Saturn / Neo Geo pads talk their native protocol; the FPGA decodes it directly.
