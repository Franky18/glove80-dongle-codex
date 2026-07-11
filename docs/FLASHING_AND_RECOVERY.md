# Flashing and recovery

The full first-installation procedure changes the split topology and deletes
Bluetooth bonds and other persistent settings. Routine feature updates normally
need only the affected normal firmware images and must not use settings-reset
images. Keep a known-good standard Glove80 firmware available before starting.

## Required files

```text
settings_reset-dongle.uf2
settings_reset-glove80_lh.uf2
settings_reset-glove80_rh.uf2
glove80_dongle-central.uf2
glove80_lh-peripheral.uf2
glove80_rh-peripheral.uf2
```

Use one matched build. Do not mix files from different source revisions.

## Entering bootloader mode

### Makerdiary dongle

Factory path:

1. Unplug the dongle.
2. Hold its button while inserting it into the computer.
3. Release after connection; `UF2BOOT` should mount and the LED should be green.

If application firmware has configured the button as reset, plug the dongle in
normally and press the button twice within 500 ms. Makerdiary documents this as
the double-reset DFU path.

A single press while the normal firmware is running performs a hardware reset:
USB disconnects briefly, the dongle restarts, and the halves reconnect. It does
not erase bonds or settings. Holding the button while already connected keeps
the MCU in reset; use hold-while-inserting or double-reset for UF2 mode.

### Glove80 left half

1. Power the half off and connect it directly with a USB-C data cable.
2. Hold the physical `C6R6 + C3R3` positions (`Magic + E` on the default layout).
3. Power it on while holding the keys.
4. Release when `GLV80LHBOOT` mounts.

### Glove80 right half

1. Power the half off and connect it directly with a USB-C data cable.
2. Hold the right-half physical `C6R6 + C3R3` positions (`I + PgDn` on the
   default layout).
3. Power it on while holding the keys.
4. Release when `GLV80RHBOOT` mounts.

The power-on methods use physical matrix positions and do not depend on a
working keymap.

## First installation or topology change

Keep devices that are not currently being flashed powered off or unplugged.
This prevents premature bonding with old settings.

### Phase 1: clear all stored bonds and settings

1. Flash `settings_reset-dongle.uf2` to the dongle, wait about 10 seconds after
   the volume disappears, then unplug it.
2. Flash `settings_reset-glove80_lh.uf2` to the left half, wait, then power it
   off and unplug it.
3. Flash `settings_reset-glove80_rh.uf2` to the right half, wait, then power it
   off and unplug it.

Settings-reset images are not usable keyboard firmware.

### Phase 2: flash normal firmware

1. Flash `glove80_dongle-central.uf2`, wait, then unplug the dongle.
2. Flash `glove80_lh-peripheral.uf2`, wait, then power off and unplug the left
   half.
3. Flash `glove80_rh-peripheral.uf2`, wait, then power off and unplug the right
   half.

### Phase 3: start and pair

1. Keep both halves off.
2. Insert the dongle normally without pressing its button and wait about 10
   seconds.
3. Power on the left half and wait up to 30 seconds.
4. Power on the right half and wait up to 30 seconds.
5. Test input from both halves.

This controlled order establishes split source 0 as left and source 1 as right,
which is the mapping used by Battery Center and Magic status.

The halves pair internally with the dongle. Do not search for them in the
computer's Bluetooth settings; the computer receives USB HID from the dongle.

## UF2 copy behavior

UF2 devices often reboot and unmount before Finder receives its final copy
acknowledgement. If Finder reports a copy error but the bootloader volume
disappears immediately, the flash may still have completed. Re-enter the
bootloader before retrying blindly.

Windows Explorer behavior will be documented after it is tested on the target
Windows machine. Until then, verify that the UF2 volume disappears and the
device restarts rather than relying only on the copy dialog.

## Troubleshooting

- No `UF2BOOT` after hold-on-insert: try the dongle double-reset path, a direct
  USB port, and a different data-capable connection.
- One half does not type: verify all three normal images came from one build and
  reset bonds on all three devices before retrying.
- Dongle types but a half does not reconnect after restart: power off both
  halves, insert the dongle first, then start left and right in that order.
- Left RGB is dark: verify that `glove80_lh-peripheral.uf2` came from the current
  complete build; ordinary left RGB is enabled in the validated configuration.
- Magic shows six red LEDs for both battery rows: confirm both halves are
  connected before tapping Magic.
- Keyboard stops working when the dongle is removed: expected; the dongle owns
  the central role.

## Reverting to stock topology

1. Obtain a known-good matched standard Glove80 firmware where the left half is
   central and the right half is peripheral.
2. Power off and disconnect the dongle.
3. Clear stored settings on both halves. Resetting only one half leaves stale
   split bonds on the other.
4. Flash the matched standard firmware to both halves following MoErgo's normal
   update procedure.
5. Start the left half, then the right half, and re-pair host Bluetooth profiles
   as needed.

References:

- <https://docs.moergo.com/glove80-user-guide/customizing-key-layout/>
- <https://docs.moergo.com/glove80-user-guide/troubleshooting/>
- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/programming/uf2boot/>
