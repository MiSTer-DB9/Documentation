# MiSTer-DB9 Documentation

End-user documentation for the MiSTer-DB9 fork of [MiSTer FPGA](https://github.com/MiSTer-devel/Main_MiSTer/wiki). The fork adds zero-latency direct support for original retro controllers connected through a DB9, DB15, or Saturn adapter to the DE10-Nano `USER_IO` port.

This repo holds the **[user-guide/](user-guide/)** — for people running MiSTer-DB9 on their hardware. How to enable controller support in the OSD, what controllers and adapters work, where to drop the optional `db9pro.key`, and how to recognise common problems.

It is published as a website at <https://mister-db9.github.io/>.

## What MiSTer-DB9 adds on top of MiSTer FPGA

| Feature | Always-free | Key-gated |
|---|---|---|
| DB9MD (Sega Genesis / Mega Drive 3-button & 6-button) | yes | — |
| DB15 (Antonio Villena splitter, Neo Geo / Supergun layout) | yes | — |
| SNAC8 (eight-pin direct controller, per-core) | yes | — |
| OSD navigation from the DB9 controller (Start+C / A / B) | yes | — |
| Saturn 1P + 2P adapter (74HC157D mux) digital pad | — | yes (`db9pro.key`) |
| 3D Control Pad (Saturn) detection on non-Saturn cores | — | yes |
| MT32-pi anti-contention double gate | — | yes |

"Key-gated" features ship in every RBF but stay inert until a valid `db9pro.key` is present at `/media/fat/db9pro.key`. See [user-guide/pro-key.md](user-guide/pro-key.md).

## Licence

GPLv3 (see [LICENSE](LICENSE)), matching `Main_MiSTer`, `Forks_MiSTer`, and `Distribution_MiSTer` in the fork.
