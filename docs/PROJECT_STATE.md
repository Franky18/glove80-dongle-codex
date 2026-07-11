# Project state

Last updated: 2026-07-11

## Current status

The dongle-central topology is working on physical hardware:

```text
Glove80 left peripheral  -- BLE --\
                                  >-- MDK USB dongle central -- USB -- computer
Glove80 right peripheral -- BLE --/
```

Confirmed behavior:

- Keystrokes from both halves reach the dongle.
- The dongle presents working USB keyboard input to the computer.
- All three devices pair and reconnect after the controlled settings-reset and
  startup sequence.
- ZMK Battery Center reports both peripheral battery values.
- Ordinary RGB works on both halves.
- A Magic tap shows stock-style left and right battery rows on the left half.
- The complete Magic status display works: first six layers, Caps Lock, Num
  Lock, Scroll Lock, four host BLE profiles, USB state, and output fallback.
- The status overlay fades out after ten seconds and restores ordinary RGB.

The user has reported the complete feature set working on hardware.

## Stable source and CI

- Repository: <https://github.com/Franky18/glove80-dongle-codex>
- Complete Magic pull request:
  <https://github.com/Franky18/glove80-dongle-codex/pull/5>
- Feature source commit: `9d42a5f24de3b4b362d9a06e4f86cdd36f79d113`
- Main merge commit: `d444c8e1128b26575b770120b68441d5d157b194`
- Artifact-producing Actions run:
  <https://github.com/Franky18/glove80-dongle-codex/actions/runs/29146909546>
- Merged artifact name: `firmware`
- The artifact contained all six expected UF2 files.

The immutable hashes and validation notes are recorded in `releases/9d42a5f/`.
The local UF2 backup is outside Git at
`../firmware/9d42a5f-full-magic-current/` relative to this repository.

## Source of truth

- Keymap: `config/my01.keymap`
- Keymap SHA-256:
  `398a93120bb9413fc7a907f9779291c2729d7315a2bf77f54a5df8dfb72b183f`
- Build matrix: `build.yaml`
- ZMK pin: `config/west.yml`
- Dongle shield: `config/boards/shields/glove80_dongle/`
- Magic integration: `patches/moergo-magic-status.patch`
- Patch loader: `CMakeLists.txt` and `zephyr/module.yml`
- GitHub workflow: `.github/workflows/build.yml`

The old non-Git local tree is preserved under the workspace `legacy/`
directory for reference only. It is not a source of truth.

## Validated development history

| Source | Result |
| --- | --- |
| `77d846e` | Initial working dongle-central topology |
| `bca895a` | Battery fetching and Battery Center proxy |
| `bc4522b` | Ordinary left-peripheral RGB restored |
| `1eb5d3f` | Magic two-row battery display |
| `9d42a5f` | Complete stock-style Magic status display |

All corresponding local UF2 checkpoints are preserved under the workspace
`firmware/` directory.

## Runtime differences from stock Glove80

- The keyboard depends on the USB dongle. Unplugging it does not restore the
  stock left-central topology.
- The computer does not pair directly with either keyboard half; the halves are
  split peripherals of the dongle.
- Host BLE profiles, output state, layer logic, and lock state live on the
  dongle central.
- Magic status is rendered physically on the left peripheral using state packed
  and sent by the dongle.
- Both halves need to be connected when Magic is tapped for both battery rows to
  be available; unavailable battery data is shown as six red LEDs.

## Windows migration status

Windows is the user's primary keyboard platform, but migration work has not yet
started. The repository and local workspace have been reorganized in preparation:

- the source directory is now a real Git clone tracking `main`;
- firmware, downloads, hardware metadata, and legacy files are separated;
- no symlinks or case-colliding paths are used;
- GitHub Actions remains the reproducible build path.

Do not claim Windows flashing or local build procedures are validated until they
are exercised on the target Windows machine.

## Open follow-up work

- Clone or transfer the repository to the Windows machine and verify Git access.
- Decide whether Windows needs only GitHub Actions downloads or a complete local
  ZMK build toolchain.
- Document and test PowerShell-based artifact download, hash verification, and
  UF2 copy steps.
- Create a durable GitHub Release for the current six validated UF2 files.
- Measure battery-life changes quantitatively if desired.
