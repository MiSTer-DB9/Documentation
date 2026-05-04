# SNAC8 (per-core direct controller protocols)

SNAC stands for "Serial / Native Adapter for Controllers". SNAC8 is the eight-pin variant: the FPGA gets raw access to all eight `USER_IO[7:0]` pins and the core itself drives the original-console protocol on those pins.

Unlike DB9MD / DB15 / Saturn, SNAC8 is **not** a uniform path through the MiSTer-DB9 fork — every core that implements it does so on its own terms. The OSD option to enable SNAC, the wiring on the adapter, and the controller compatibility list are all per-core.

## What stays the same fork-wide

- The eight pins are routed: `USER_IO[7:0]` are all inout and all reach the core.
- `USER_IO[7]` (DB9 pin 2) is added on top of upstream's seven-pin port. Without the MiSTer-DB9 build of the core, this pin would not be available to SNAC.
- The push-pull / open-drain configuration of each pin is per-core: the core asks the fork's `USER_PP` mask to drive the pins it needs as push-pull, leaves the rest open-drain.

## What changes per core

- Which OSD entry enables SNAC (some cores call it `SNAC`, some `Pad 1 SNAC`, some have a port-pick option, some have nothing — they auto-detect).
- The cable / adapter pinout (NES SNAC ≠ SNES SNAC ≠ PCE SNAC ≠ Saturn SNAC).
- Which controllers are supported (e.g. PSX SNAC handles DualShock; SNES SNAC handles standard pads + multi-tap with caveats).

## Cores known to support a SNAC option

The set varies over time and is best read from each core's release notes or its OSD. The fork-wide commitment is just "the eight pins are wired through and you can drive them" — what each core does with them is the core's own decision.

## DB9 + SNAC8 share the same physical port

SNAC8 takes the same `USER_IO` pins that DB9MD / DB15 / Saturn use, so only one path can drive the connector at any moment. The fork enforces this **inside the FPGA**, with SNAC always winning:

- When the core's SNAC mode is enabled (via its OSD entry — name varies per core), the `UserIO Joystick` setting is silently ignored. The DB9 / DB15 / Saturn protocol module is held inert (`USER_OUT` idle, `joydb_*` outputs zero), so a DB9 controller plugged into the same port produces no inputs and no OSD ghost clicks.
- When SNAC is off, `UserIO Joystick` works as expected.

The OSD does **not** grey out the `UserIO Joystick` entry when SNAC is on — the setting is just inert. If you toggle `UserIO Joystick` while SNAC is on and nothing happens, that's why. To use a DB9 / DB15 / Saturn controller on the same port, turn SNAC off in the core's OSD first.

Before the gate landed (pre-2026-05-04), enabling both at once produced electrical conflicts on `USER_IO` pins and OSD ghost clicks via `joy_raw`. Cores affected: NES, SNES, SMS, Saturn, TurboGrafx16, Genesis, MegaCD, S32X, PSX, Atari7800, SGB.
