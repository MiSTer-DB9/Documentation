# Define DB9 Buttons

Every core lets you **remap** which physical controller button does what, the
same way the USB "Define buttons" page works for USB pads — but for your DB9 /
DB15 / Saturn controller. The mapping is saved **per core and per controller
type**, so each game keeps its own layout.

You don't have to do anything to get a sensible default: each core ships a
factory layout matched to its own button names, so most controllers just work
out of the box. Use this page only when you want to move a button.

## Opening the page

1. Set `UserIO Joystick` to your controller type and save (see
   [OSD Menu](osd-menu.md)).
2. Open the OSD and find the **Define DB9 buttons** entry (in the same area as
   the other controller options).
3. Select it. The page walks you through the buttons one at a time.

## Defining a layout

The page shows one button name at a time (e.g. `Press: A`). For each one:

| You do | Result |
|---|---|
| **Press the controller button** you want for that action | Maps it, moves to the next |
| **Start + C** (tap, quick) | Leaves that action **unmapped**, moves on |
| **Start + C** (hold ~1 s) | **Cancels** — restores the previously saved layout and exits |
| Board **User** button | Same as a quick tap — unmap this action |
| **Enter** (USB keyboard) | **Finish** — saves what you've set so far |
| **F12** (USB keyboard) | **Clear all** — resets to the factory default and starts over (shows a brief "Clearing" message) |

The D-pad (up/down/left/right) is fixed and not part of the walk — those are
dedicated pins and are never remapped.

When you've pressed a button for every action (or pressed **Enter** to finish
early), the layout is saved automatically. It loads again next time you start
that core with that controller type.

## Notes per controller type

- **DB9MD (Mega Drive):** has no L/R shoulder buttons, so a core that asks for
  L/R (e.g. some handheld cores) leaves those unmapped on a Mega Drive pad —
  press another button for them in this page if you need them. A 3-button pad
  can reach only Start, A/B/C and the D-pad.
- **DB15 (NeoGeo / Supergun):** six face buttons available.
- **Saturn:** all six face buttons plus L/R triggers. The pad has no `Select`, so
  cores that need `Select`/`Mode` use the **Start + B** chord (or the `R` trigger
  when the core doesn't otherwise use it).

## Resetting

Open the page and press **F12** (Clear all) to drop back to the core's factory
default at any time.
