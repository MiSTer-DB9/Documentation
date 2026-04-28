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

Because SNAC8 takes the same `USER_IO` pins that DB9MD / DB15 / Saturn use, only one of the two paths can be active at any moment. The OSD makes them mutually exclusive: turning on `UserIO Joystick = DB9MD` (or DB15 or Saturn) implicitly disables SNAC mode for that core, and vice versa.

If you connect a SNAC adapter to the DE10-Nano while `UserIO Joystick` is set to a DB9 mode, the DB9 module will drive its protocol onto pins the SNAC adapter doesn't expect — usually nothing breaks, but you also won't get useful inputs. Pick one or the other, save settings, reboot the core if needed.
