# Installation

To use DB9, DB15, SNAC8, or Saturn controllers, the MiSTer FPGA on the SD card must be the MiSTer-DB9 build, not the stock upstream build. The same applies to the on-SD menu.

## Files to replace

Two files on the SD card root must be the MiSTer-DB9 versions:

| File | Source |
|---|---|
| `MiSTer` (the Linux ARM binary) | `https://github.com/MiSTer-DB9/Main_MiSTer/tree/master/releases` (latest release) |
| `menu.rbf` (the menu core) | `https://github.com/MiSTer-DB9/Menu_MiSTer/tree/master/releases` (latest release) |

The cores themselves (`<core>.rbf` files in `_<system>/` directories) are also DB9-built and ship from each per-core fork repository under the `MiSTer-DB9` GitHub organisation. The recommended way to keep them up to date is the updater script described below.

## Updater script

To keep every core in sync with the DB9 build (falling back to the upstream build when no DB9 build exists), drop the updater script at `/Scripts/update.sh` on the SD card:

```
https://raw.githubusercontent.com/theypsilon/Updater_script_MiSTer_DB9/master/update.sh
```

When you run **Update** from the MiSTer Scripts menu, the script downloads each core: it picks the MiSTer-DB9 build if one exists for that core, otherwise the upstream build. The first run is slow (downloads everything); subsequent runs are incremental.

## Boot sequence to expect

1. Power on the DE10-Nano.
2. The DB9 build of `MiSTer` starts.
3. The menu core (`menu.rbf`) loads. By default the "UserIO Joystick" option is **off** — the DB controller is not active until you enable it. See [osd-menu.md](osd-menu.md).
4. Pick a core. The same OSD option needs to be set per-core, then saved.

## What to do after the first boot

- For each core you want to use a DB controller with: open the OSD (F12 on a USB keyboard or `Start+C` on the DB9 pad after enabling), set **UserIO Joystick** to your controller type, set **UserIO Players** to 1 or 2, then **Save settings**.
- The setting is per-core — the menu core, Mega Drive, Neo Geo, Saturn, and so on each remember their own value.

## What you get by default (and how to opt in)

If you have **not** set a `filter` for the ENCC database in your
`downloader.ini`, you get the full stable set **minus** three opt-in
channels:

- `unstable` — nightly builds off upstream HEAD (for testing only).
- `jtbeta` — jotego's beta game variants (need jotego's patreon `beta.bin`
  key; useless without it).
- `dualsdram` — Dual-SDRAM core variants (PSX, Saturn).

If you already set your own `filter` (e.g. `filter = console`), nothing
changes — those channels were already excluded for you.

To pull a channel in, add its token to your `filter`. The Downloader rule:
**a filter with any positive (non-`!`) term becomes an allow-list** — only
files carrying that tag are downloaded. So:

- `filter = unstable` — downloads **only** unstable cores, nothing else
  (no stable cores, no menu). Use this only for an isolated unstable test.
- `filter = !jtbeta !dualsdram` — keeps the **whole normal system and adds
  unstable** (you just drop the default `!unstable`). This is the right
  choice for testing an unstable core while keeping a working setup; pick
  the `_Unstable/...` build in the core's OSD.
- `filter = unstable-saturn` — only the Saturn core's unstable build
  (allow-list, that core only). Replace `saturn` with the core slug.
- `filter = all` — everything, including unstable, beta and Dual-SDRAM.

After changing `filter`, run **Update** again.
