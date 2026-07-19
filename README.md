# Glove80 + Makerdiary nRF52840 USB Dongle

[![Build Glove80 dongle firmware](https://github.com/Franky18/glove80-dongle-codex/actions/workflows/build.yml/badge.svg)](https://github.com/Franky18/glove80-dongle-codex/actions/workflows/build.yml)

This repository turns a Makerdiary nRF52840 MDK USB Dongle into the wired USB
central for a Glove80. Both keyboard halves connect to the dongle over BLE, and
the computer sees a normal USB keyboard.

```text
Glove80 left peripheral  -- BLE --\
                                  >-- MDK USB dongle central -- USB -- computer
Glove80 right peripheral -- BLE --/
```

The repository is designed to be forked. Replace one keymap file in your fork,
let GitHub Actions build a matched six-file firmware set, and follow the
first-installation procedure to move the split central role to the dongle.

## Hardware compatibility warning

The flash layout in this repository has only been validated with this exact
device profile:

| Property | Validated value |
| --- | --- |
| Device | Makerdiary nRF52840 MDK USB Dongle |
| Board ID | `nRF52840-MDK-USB-DONGLE` |
| UF2 bootloader | `0.7.1`, dated 2023-07-20 |
| SoftDevice | not present |
| Application address | `0x00001000` |
| UF2 family ID | `0xADA52840` |

Do not flash a generic nRF52840 dongle, a dongle with a different flash layout,
or a device whose `INFO_UF2.TXT` does not match the validated profile. See
[Hardware profile](docs/HARDWARE_PROFILE.md) before building or flashing.

## Quick start: build with your keymap

1. [Fork this repository](https://github.com/Franky18/glove80-dongle-codex/fork).
2. Open the **Actions** tab in your fork and enable workflows if GitHub asks.
3. Export your layout from the MoErgo Glove80 Layout Editor.
4. In your fork, replace `config/glove80.keymap` with the exported file. Keep
   the filename exactly `glove80.keymap`.
5. Commit the change to your fork. The build workflow runs on every push, pull
   request, and manual dispatch.
6. Open the completed **Build Glove80 dongle firmware** run. Require both
   `Validate public configuration` and all six firmware builds to pass.
7. Download the merged artifact named `firmware`.

The committed `config/glove80.keymap` is the **Glove80 Factory Default Layout**
exported by the official MoErgo Layout Editor. It gives forks a neutral,
buildable starting point. Replace it with your own Layout Editor export when you
want a different layout.

The artifact must contain exactly these files:

| File | Device and purpose |
| --- | --- |
| `glove80_dongle-central.uf2` | Normal firmware for the Makerdiary dongle central |
| `glove80_lh-peripheral.uf2` | Normal firmware for the Glove80 left peripheral |
| `glove80_rh-peripheral.uf2` | Normal firmware for the Glove80 right peripheral |
| `settings_reset-dongle.uf2` | Clears dongle settings and bonds |
| `settings_reset-glove80_lh.uf2` | Clears left-half settings and bonds |
| `settings_reset-glove80_rh.uf2` | Clears right-half settings and bonds |

Do not combine files from different workflow runs. Do not flash anything if a
job failed or one of the six files is missing.

Read [Custom keymaps](docs/CUSTOM_KEYMAP.md) for keymap-specific behavior and
[Build and release](docs/BUILD_AND_RELEASE.md) for the complete build workflow.

## First installation

Moving the central role from the Glove80 left half to the dongle requires
clearing old split bonds on all three devices. Keep a known-good stock Glove80
firmware set before starting.

At a high level:

1. Flash the matching settings-reset image to the dongle, left half, and right
   half. A settings-reset image is not usable keyboard firmware.
2. Flash the three normal images from the same build.
3. Start the dongle first, then the left half, then the right half.

The order matters: it establishes the left half as split source 0 and the right
half as source 1 for battery and Magic status. Review the complete
[Flashing and recovery](docs/FLASHING_AND_RECOVERY.md) procedure before writing
any UF2 file.

## Important behavior differences

- The keyboard depends on the dongle while this topology is installed.
- The Glove80 halves pair internally with the dongle, not with the computer.
- The keymap, layers, output selection, host profiles, and lock state run on the
  dongle central.
- `OUT_USB` selects the dongle's USB connection. Host BLE profile keys manage
  profiles stored on the dongle.
- RGB commands are relayed to both halves. The dongle uses a logical one-pixel
  RGB device to own central RGB state; no physical LED is attached to it.
- A standard Magic status action asks the dongle for both peripheral battery
  levels and renders the stock-style status display on the left half. A custom
  keymap that removes the Magic/status action will not expose that display.
- A keymap `&bootloader` action runs on the dongle central. Use the documented
  physical bootloader entry method when flashing either Glove80 half.

## Documentation

- [Custom keymaps](docs/CUSTOM_KEYMAP.md)
- [Flashing and recovery](docs/FLASHING_AND_RECOVERY.md)
- [Hardware profile and compatibility](docs/HARDWARE_PROFILE.md)
- [Build and release](docs/BUILD_AND_RELEASE.md)
- [Architecture and design decisions](docs/ARCHITECTURE_AND_DECISIONS.md)
- [Hardware-validated project state](docs/PROJECT_STATE.md)
- [Contributing](CONTRIBUTING.md)

## Support and limitations

Before opening an issue, confirm that all firmware files came from one build,
record the source commit and Actions run URL, and collect the dongle's
`INFO_UF2.TXT`. The issue template asks for the hardware, host, reset, pairing,
and Bluetooth-bond information needed to investigate safely.

This is a community project, not an official MoErgo or Makerdiary product.

## License and acknowledgements

This repository is licensed under the [MIT License](LICENSE). It builds on ZMK,
the MoErgo ZMK tree, the Glove80 Layout Editor output format, and Makerdiary's
nRF52840 MDK USB Dongle support. Upstream copyright and SPDX notices remain in
the files derived from those projects.
