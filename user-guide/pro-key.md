# The `db9pro.key` File

A small subset of MiSTer-DB9 features is **key-gated**. They ship in every RBF but stay inert until a valid `db9pro.key` is found at `/media/fat/db9pro.key`. Without the file, those features behave as if they were absent: no Saturn option in the OSD acts on input, and so on. Currently only Saturn is key-gated; other DB9-fork features (DB9MD, DB15, SNAC8, MT32-pi anti-contention) are always-free.

## What unlocks with the key

| Feature | Without key | With valid key |
|---|---|---|
| DB9MD (Mega Drive 3/6-button) | works | works (no change) |
| DB15 (Antonio Villena splitter) | works | works (no change) |
| SNAC8 | works | works (no change) |
| **Saturn 1P / 2P adapter (digital pad)** | option in OSD does nothing | works |
| **3D Control Pad detection** (analog switch position) | option in OSD does nothing | detection works (digital play needs SNAC mode in the Saturn core) |
| MT32-pi anti-contention double gate | always active | always active (not key-gated) |

## Where to put the file

- Path: `/media/fat/db9pro.key`
- Size: 128 bytes (do not edit by hand — the file carries a signature).
- Permissions: any user-readable mode.

If the file is missing, the wrong size, or the signature does not verify, the gate stays locked. There is no error popup — the OSD options that depend on the key simply do nothing.

## What the file contains

The file binds an unlock to a specific identity (so leaked keys are traceable) and a specific time window (so leaks have a finite lifetime). The key carries:

| Field | Purpose |
|---|---|
| `magic` (`DB9K`) | Identifies the file as a MiSTer-DB9 key. |
| Version | Reserved for future format changes. |
| `customer_id` | Customer identity. |
| `issue` / `expiry` | Validity window (typically 60–90 days). |
| `feature_mask` | Which features this key unlocks. Bit 0 = Saturn; bits 1..31 reserved for future gated features. |
| Per-customer seed | Used by the FPGA challenge-response. |
| Signature | Ed25519 signature of all of the above. |

If you are curious about how the FPGA validates the file, see [the key-gate design notes](https://github.com/MiSTer-DB9/Documentation-Maintainer/blob/main/maintainer-guide/key-gate-design.md). End users do not need to know that detail.

## Renewing an expired key

When the expiry passes, the FPGA gate locks again on the next boot. You will see the same "OSD option does nothing" symptom you had before installing a key. To renew:

1. Get a new key file through the same channel that issued the original (the project ships keys to active customers; renewal is automatic if you stay subscribed).
2. Replace `/media/fat/db9pro.key` with the new file.
3. Reboot or load the core again — the gate negotiates fresh on every core load.

## When the gate stays locked even with a key present

In order, check:

1. Is the file exactly 128 bytes? (`ls -l /media/fat/db9pro.key`)
2. Is the SD card real-time clock close to current? An RTC reset to 1970 can fail the validity check.
3. Is the MiSTer build the MiSTer-DB9 build, not stock upstream? See [installation.md](installation.md).
4. Is the core itself a MiSTer-DB9 build? Stock upstream RBFs do not contain the gate logic.

The gate is intentionally fail-closed: any unexpected condition keeps Saturn locked. There is no override.
