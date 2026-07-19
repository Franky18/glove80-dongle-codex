# Custom keymaps

This repository has one user-editable keymap source:

```text
config/glove80.keymap
```

The Glove80 left and right builds select that file directly. The dongle build
selects `config/glove80_dongle.keymap`, which includes the same
`glove80.keymap`. This keeps all three normal firmware images on one keymap
definition while the keymap logic executes on the dongle central.

## Replace the example with your layout

1. Fork the repository.
2. Export your layout from the MoErgo Glove80 Layout Editor.
3. Rename the exported file to `glove80.keymap` if necessary.
4. Replace `config/glove80.keymap` in your fork. Do not paste the export into
   `config/glove80_dongle.keymap`.
5. Commit the change and wait for the GitHub Actions build.
6. Download the merged `firmware` artifact only after validation and all six
   firmware jobs pass.

The keymap committed to the upstream repository is the **Glove80 Factory Default
Layout** exported by the official MoErgo Layout Editor. It is a neutral,
buildable starting point. Replace it only when you want to build a different
layout.

## What runs on the dongle

The dongle is the ZMK central, so it owns:

- layer and behavior state;
- macros, combos, hold-taps, and tap dances;
- USB and host BLE output selection;
- host BLE profiles and bonds;
- Caps Lock, Num Lock, and Scroll Lock state;
- global RGB command state;
- the Magic status payload sent to the left peripheral.

The Glove80 halves scan their physical matrices and send key positions to the
dongle. They also render ordinary RGB, and the left half renders the transported
Magic status display.

## Feature compatibility

Most Layout Editor exports can replace the example without changes, provided
they target the MoErgo ZMK generation used by the pinned revision in
`config/west.yml`.

Review these behaviors in your exported keymap:

| Keymap feature | Dongle-topology behavior |
| --- | --- |
| Normal keys, layers, macros, combos | Run on the dongle central |
| `OUT_USB` | Selects the dongle's USB output |
| `OUT_BLE` and BLE profile keys | Select host BLE profiles stored on the dongle |
| RGB actions | Update dongle-owned RGB state and relay it to both halves |
| Magic/status action | Requests and transports central status to the left half |
| `&bootloader` | Enters the dongle bootloader, not a half's bootloader |

Removing the Magic/status action does not prevent normal typing, but the
stock-style Magic status presentation will no longer be accessible from that
keymap. Removing RGB controls does not disable the compiled RGB support; it only
removes those controls from the layout.

## Files a normal keymap user should not edit

Changing a layout does not require edits to:

- `build.yaml`;
- `config/west.yml`;
- `.github/workflows/build.yml`;
- `config/boards/shields/glove80_dongle/`;
- `patches/moergo-magic-status.patch`;
- `CMakeLists.txt` or `zephyr/module.yml`.

Those files define the tested hardware, split topology, pinned ZMK integration,
and six-image build. Treat changes to them as firmware-development work rather
than keymap customization.

## Updating installed firmware

Settings-reset images are for the first topology change or intentional recovery.
Do not flash them for an ordinary keymap update.

The keymap executes on the dongle, but the supported public update workflow is
to keep versions matched by flashing all three normal images from one successful
build. Follow the routine update section in
[Flashing and recovery](FLASHING_AND_RECOVERY.md).
