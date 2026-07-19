# Project state

Last updated: 2026-07-14

## Current status

The dongle-central topology is working on physical hardware:

```text
Glove80 left peripheral  -- BLE --\
                                  >-- MDK USB dongle central -- USB -- computer
Glove80 right peripheral -- BLE --/
```

Confirmed behavior on the recorded hardware profile:

- keystrokes from both halves reach the dongle;
- the dongle presents working USB keyboard input to the computer;
- all three devices pair and reconnect after the controlled reset and startup
  sequence;
- ZMK Battery Center reports both peripheral battery values;
- ordinary RGB works on both halves;
- a Magic tap shows stock-style left and right battery rows on the left half;
- the complete Magic status display works for layers, locks, BLE profiles, USB,
  and output fallback;
- the status overlay fades out and restores ordinary RGB.

This validation applies to the recorded hardware-validated release, not
automatically to firmware built from an arbitrary fork or custom keymap.

## Last hardware-validated release

- Repository: <https://github.com/Franky18/glove80-dongle-codex>
- Feature source commit: `9d42a5f24de3b4b362d9a06e4f86cdd36f79d113`
- Main merge commit: `d444c8e1128b26575b770120b68441d5d157b194`
- Artifact-producing Actions run:
  <https://github.com/Franky18/glove80-dongle-codex/actions/runs/29146909546>
- Merged artifact name: `firmware`
- Immutable hashes and validation notes: `releases/9d42a5f/`

The public-repository documentation and keymap entry-point cleanup after that
release do not retroactively change its provenance. A later build must not be
called hardware-validated until it has its own recorded test result.

## Public fork contract

The repository now exposes one normal customization point:

```text
config/glove80.keymap
```

Fork users replace that file and build through GitHub Actions. The dongle entry
point includes the same keymap, while the six-image matrix, exact hardware
profile, pinned MoErgo ZMK revision, and Magic integration remain
repository-controlled.

The public default is the official Layout Editor export named **Glove80 Factory
Default Layout**. Its SHA-256 is:

```text
b5e16f21609982e8fdc9535b90ce5169252a4bbf80780a451d7172a266081e51
```

## Source of truth

- User keymap: `config/glove80.keymap`
- Dongle keymap wrapper: `config/glove80_dongle.keymap`
- Build matrix: `build.yaml`
- ZMK pin: `config/west.yml`
- Dongle shield: `config/boards/shields/glove80_dongle/`
- Magic integration: `patches/moergo-magic-status.patch`
- Patch loader: `CMakeLists.txt` and `zephyr/module.yml`
- GitHub workflow: `.github/workflows/build.yml`
- Hardware compatibility gate: `docs/HARDWARE_PROFILE.md`

## Validated development history

| Source | Result |
| --- | --- |
| `77d846e` | Initial working dongle-central topology |
| `bca895a` | Battery fetching and Battery Center proxy |
| `bc4522b` | Ordinary left-peripheral RGB restored |
| `1eb5d3f` | Magic two-row battery display |
| `9d42a5f` | Complete stock-style Magic status display |

Historical release records are immutable and remain under `releases/`.

## Runtime differences from stock Glove80

- The keyboard depends on the USB dongle. Unplugging it does not restore the
  stock left-central topology.
- The computer does not pair directly with either keyboard half; the halves are
  split peripherals of the dongle.
- Host BLE profiles, output state, layer logic, and lock state live on the
  dongle central.
- Magic status is rendered physically on the left peripheral using state packed
  and sent by the dongle.
- Both halves must be connected when Magic is tapped for both battery rows to be
  available; unavailable battery data is shown as six red LEDs.

## Windows and local build status

GitHub Actions is the supported public build path on Windows, macOS, and Linux.
It requires no local ZMK toolchain. A portable repo-relative local build helper
is not currently included and must be tested from a clean checkout before being
documented as a supported path.

## Open follow-up work

- run the complete six-target CI after the public-repository changes;
- publish the current validated six-file set as durable GitHub Release assets;
- add a repo-relative local Docker helper only after testing it from a clean
  checkout;
- continue collecting Windows adapter and stale host-bond reports;
- measure battery-life changes quantitatively if desired.
