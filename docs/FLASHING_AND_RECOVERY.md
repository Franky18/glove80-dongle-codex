# Flashing and recovery

The first installation changes the Glove80 split topology and deletes Bluetooth
bonds and other persistent settings. Routine firmware or keymap updates do not
need a settings reset.

Before starting:

- verify the dongle against [Hardware profile and compatibility](HARDWARE_PROFILE.md);
- keep a known-good matched stock Glove80 firmware set;
- download one successful six-file artifact and do not mix build revisions;
- keep devices that are not currently being flashed powered off or unplugged.

## Firmware files

| File | Physical device | Use |
| --- | --- | --- |
| `glove80_dongle-central.uf2` | Makerdiary dongle | Normal firmware |
| `glove80_lh-peripheral.uf2` | Glove80 left | Normal firmware |
| `glove80_rh-peripheral.uf2` | Glove80 right | Normal firmware |
| `settings_reset-dongle.uf2` | Makerdiary dongle | Clear settings and bonds |
| `settings_reset-glove80_lh.uf2` | Glove80 left | Clear settings and bonds |
| `settings_reset-glove80_rh.uf2` | Glove80 right | Clear settings and bonds |

Settings-reset images are temporary maintenance firmware. They cannot be used as
a keyboard and must be followed by the matching normal image.

## Entering bootloader mode

### Makerdiary dongle

Factory path:

1. Unplug the dongle.
2. Hold its button while inserting it into the computer.
3. Release after connection; `UF2BOOT` should mount and the LED should be green.

If application firmware has configured the button as reset, plug the dongle in
normally and press the button twice within 500 ms. Makerdiary documents this as
the double-reset DFU path.

A single press while normal firmware is running performs a hardware reset: USB
disconnects briefly, the dongle restarts, and the halves reconnect. It does not
erase bonds or settings. Holding the button while already connected keeps the
MCU in reset; use hold-while-inserting or double-reset for UF2 mode.

### Glove80 left half

1. Power the half off and connect it directly with a USB-C data cable.
2. Hold physical positions `C6R6 + C3R3` (`Magic + E` on the default layout).
3. Power it on while holding the keys.
4. Release when `GLV80LHBOOT` mounts.

### Glove80 right half

1. Power the half off and connect it directly with a USB-C data cable.
2. Hold the right-half physical positions `C6R6 + C3R3` (`I + PgDn` on the
   default layout).
3. Power it on while holding the keys.
4. Release when `GLV80RHBOOT` mounts.

These power-on methods use physical matrix positions and do not depend on the
installed keymap. A keymap `&bootloader` action executes on the dongle central
and must not be used as the only recovery path for either half.

## First installation or topology change

Use this complete procedure when changing from the stock left-central topology
to the dongle-central topology, or when intentionally rebuilding all split bonds.

### Phase 1: clear stored state on all three devices

1. Flash `settings_reset-dongle.uf2` to the dongle, wait about 10 seconds after
   the volume disappears, then unplug it.
2. Flash `settings_reset-glove80_lh.uf2` to the left half, wait, then power it
   off and unplug it.
3. Flash `settings_reset-glove80_rh.uf2` to the right half, wait, then power it
   off and unplug it.

Resetting only one or two devices leaves stale split bonds on the others.

### Phase 2: flash normal firmware

1. Flash `glove80_dongle-central.uf2`, wait, then unplug the dongle.
2. Flash `glove80_lh-peripheral.uf2`, wait, then power off and unplug the left
   half.
3. Flash `glove80_rh-peripheral.uf2`, wait, then power off and unplug the right
   half.

### Phase 3: start and pair in a controlled order

1. Keep both halves off.
2. Insert the dongle normally without pressing its button and wait about 10
   seconds.
3. Power on the left half and wait up to 30 seconds.
4. Power on the right half and wait up to 30 seconds.
5. Test input from both halves.

This order establishes split source 0 as left and source 1 as right. Battery
Center and Magic status depend on that mapping.

The halves pair internally with the dongle. Do not search for either half in the
computer's Bluetooth settings; the normal host path is USB HID from the dongle.

## Routine firmware or keymap update

Do not use any `settings_reset-*.uf2` file for an ordinary update. Preserving NVS
keeps the existing split bonds and host profiles.

For the supported public workflow:

1. Download all files from one successful build.
2. Keep devices not currently being flashed powered off or disconnected.
3. Flash `glove80_dongle-central.uf2`.
4. Flash `glove80_lh-peripheral.uf2`.
5. Flash `glove80_rh-peripheral.uf2`.
6. Start the dongle, left half, and right half in that order.

The keymap logic is central-owned, but flashing the three matched normal images
keeps the ZMK revision, split protocol, and Magic integration aligned for public
support.

## UF2 copy behavior

UF2 devices often reboot and unmount before an operating system receives its
final copy acknowledgement. If a copy dialog reports an error but the
bootloader volume disappears immediately, the flash may still have completed.
Wait for the device to restart and re-enter the bootloader before retrying.

Never retry blindly with a file intended for another physical device.

## Windows Bluetooth bond warning

The dongle can perform two BLE roles at once:

- BLE central for the two Glove80 halves;
- optional BLE peripheral to Windows for a host profile or software such as ZMK
  Battery Center.

Those are separate relationships. The halves' split bonds are stored on the
three keyboard devices; a Windows `Glove80 Dongle` entry is a host bond.

A stale Windows host bond has been observed to trigger repeated reconnect and
disconnect activity that interferes with split stability. If you use USB only:

1. remove the `Glove80 Dongle` entry from Windows Bluetooth settings;
2. restart the dongle, then the left and right halves in order;
3. confirm stable USB typing before investigating radio range or interference.

If host BLE or Battery Center access is intentional, remove the stale Windows
entry and pair it again only after the split link is stable. Clearing the Windows
entry does not erase the internal dongle-to-half split bonds.

## Troubleshooting

- **No `UF2BOOT` after hold-on-insert:** try double-reset, a direct USB port,
  and a different data-capable connection. Re-check the hardware profile.
- **One half does not type after first installation:** verify all six files came
  from one build, then repeat the complete three-device reset and controlled
  pairing procedure.
- **A half does not reconnect after restart:** power off both halves, insert the
  dongle first, then start left and right in order.
- **Repeated reconnects on Windows:** inspect and remove a stale host Bluetooth
  bond before treating the problem as RF-only.
- **Left RGB is dark:** verify the left image came from the current complete
  build; ordinary left-peripheral RGB is enabled in the validated configuration.
- **Magic shows six red LEDs for a battery row:** confirm the corresponding half
  is connected when Magic is tapped.
- **Magic is unavailable with a custom layout:** confirm the exported keymap
  still contains a Magic/status action.
- **Keyboard stops when the dongle is removed:** expected; the dongle owns the
  central role.

When opening an issue, include the source commit, Actions run URL,
`INFO_UF2.TXT`, operating system and Bluetooth adapter, exact files flashed,
reset sequence, startup order, and whether the host has a dongle BLE bond.

## Reverting to the stock topology

1. Obtain a known-good matched standard Glove80 firmware where the left half is
   central and the right half is peripheral.
2. Power off and disconnect the dongle.
3. Clear stored settings on both halves. Resetting only one half leaves a stale
   split bond on the other.
4. Flash the matched standard firmware to both halves following MoErgo's normal
   update procedure.
5. Start the left half, then the right half, and re-pair host Bluetooth profiles
   as needed.

References:

- <https://docs.moergo.com/glove80-user-guide/customizing-key-layout/>
- <https://docs.moergo.com/glove80-user-guide/troubleshooting/>
- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/programming/uf2boot/>
