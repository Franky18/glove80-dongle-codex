# Glove80 + Makerdiary nRF52840 MDK USB Dongle

This repository builds a three-part ZMK keyboard:

```text
Glove80 left peripheral  -- BLE --\
                                  >-- MDK USB dongle central -- USB -- computer
Glove80 right peripheral -- BLE --/
```

The configuration is for the inspected dongle only:

- Model: Makerdiary nRF52840 MDK USB Dongle
- Board ID: `nRF52840-MDK-USB-DONGLE`
- UF2 Bootloader: `0.7.1` dated 2023-07-20
- SoftDevice: not present
- ZMK application address: `0x1000`
- UF2 family ID: `0xADA52840`

## Build with GitHub Actions

1. Create an empty GitHub repository.
2. Push the **contents of this directory** to the repository root. Do not push
   the parent `glove80dongle` directory as the repository root.
3. Open the repository's **Actions** tab.
4. Select **Build Glove80 dongle firmware** and wait for all six matrix jobs.
5. Download the merged artifact named `firmware`.

The archive should contain exactly these six UF2 files:

```text
glove80_dongle-central.uf2
glove80_lh-peripheral.uf2
glove80_rh-peripheral.uf2
settings_reset-dongle.uf2
settings_reset-glove80_lh.uf2
settings_reset-glove80_rh.uf2
```

Do not flash anything if any build job failed or if one of the six files is
missing.

## Keymap

`config/my01.keymap` is the canonical keymap exported from the MoErgo Layout
Editor. Both `config/glove80.keymap` and `config/glove80_dongle.keymap` include
that one file, so there is only one copy to update.

When replacing the keymap, keep the filename `config/my01.keymap` or update both
small entry-point files.

## First-time flashing overview

Changing the central from the Glove80 left half to the dongle requires clearing
the old split bonds on **all three devices**.

1. Keep a known-good stock Glove80 UF2 as a recovery image.
2. Flash the matching `settings_reset-*.uf2` to the dongle, left half, and right
   half. A settings-reset image is not usable as a keyboard.
3. Flash `glove80_dongle-central.uf2` to the dongle.
4. Flash `glove80_lh-peripheral.uf2` to the left half.
5. Flash `glove80_rh-peripheral.uf2` to the right half.
6. Leave the dongle connected to the computer. Power the left half on, wait for
   it to connect, then power the right half on.

The Glove80 halves pair internally with the dongle. Do not look for the halves
in the computer's Bluetooth settings. The computer receives USB HID reports
from the dongle.

Detailed flashing and recovery steps should be reviewed before any UF2 is
written.

## Known behavior differences

- The keyboard depends on the dongle while this firmware topology is installed.
- `OUT_USB` in the keymap selects the dongle's USB connection.
- The four Bluetooth profile keys control host BLE profiles stored on the
  dongle, not on the left half.
- RGB commands are relayed to both peripherals, and ordinary RGB effects remain
  available. A dummy one-pixel RGB device on the dongle maintains central state.
- USB Caps Lock/Num Lock/Scroll Lock indicators are forwarded to peripherals.
- Tapping Magic asks the dongle for both peripheral battery levels and shows the
  stock-style two battery rows on the left half for ten seconds. Row 3 is the
  left battery (peripheral 0); row 4 is the right battery (peripheral 1). The
  normal RGB effect resumes automatically.
- This first Magic-status version shows battery rows only. Layer, lock, BLE, and
  USB status pixels are intentionally omitted. Both halves must be connected
  when Magic is tapped; a level that the dongle cannot read is shown as six red
  LEDs.

The battery-status integration is kept as a small patch in
`patches/moergo-magic-battery-status.patch`. The root Zephyr module applies it
only to the exact MoErgo ZMK revision pinned in `config/west.yml`; a revision or
patch mismatch fails the build.

## Reverting to the stock topology

To remove the dongle, clear settings on both Glove80 halves again, then flash a
normal matched pair where `glove80_lh` is central and `glove80_rh` is peripheral.
The two halves will not work as a stock pair merely by unplugging the dongle.
