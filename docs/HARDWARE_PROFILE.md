# Hardware profile

This configuration is intentionally tied to the inspected dongle. Do not reuse
its flash map for a different nRF52840 board based only on the MCU name.

## Inspected `INFO_UF2.TXT`

```text
UF2 Bootloader 0.7.1 lib/nrfx (v2.0.0) lib/tinyusb (0.12.0-145-g9775e7691) lib/uf2 (remotes/origin/configupdate-9-gadbb8c7)
Model: Makerdiary nRF52840 MDK USB Dongle
Board-ID: nRF52840-MDK-USB-DONGLE
Date: Jul 20 2023
SoftDevice: not found
```

Derived build requirements:

- Application address: `0x00001000`
- UF2 family ID used by this working build: `0xADA52840`
- No SoftDevice partition
- Bootloader begins at `0x000F4000`

## Flash layout

| Region | Start | End | Size | Policy |
| --- | ---: | ---: | ---: | --- |
| MBR | `0x00000000` | `0x00000FFF` | 4 KiB | Read-only |
| ZMK application | `0x00001000` | `0x000D3FFF` | 844 KiB | Firmware |
| ZMK settings/NVS | `0x000D4000` | `0x000F3FFF` | 128 KiB | Persistent state |
| UF2 bootloader | `0x000F4000` | `0x000FFFFF` | 48 KiB | Read-only |

The authoritative repository definition is
`config/boards/shields/glove80_dongle/glove80_dongle_flash.dtsi`.

## Dongle-only logical devices

The MoErgo RGB implementation needs a central RGB state owner even though the
dongle has no Glove80 LED strip. The shield therefore declares:

- a one-pixel dummy WS2812 SPI strip using unused P0.13;
- a dummy external-power device using unused P0.14;
- a mock key scanner with no physical keys;
- the production Glove80 80-position matrix transform.

No LED strip or power switch is physically attached to P0.13 or P0.14.

The dummy devices own global RGB state on the central. Complete Magic state is
packed by the dongle and rendered by the physical left-half LED strip; the
dongle itself has no Glove80 status LEDs.

## UF2 entry behavior

Factory behavior is to hold the button while inserting the dongle and release
it after connection; the LED should turn green and `UF2BOOT` should mount.

After application firmware configures the button as reset, Makerdiary documents
a second path: press the button twice within 500 ms to enter DFU mode. This was
needed during the first hardware installation when hold-on-insert did not mount
the volume after the reset image had been flashed.

With the current ZMK board definition, one press is an ordinary active-low
hardware reset on P0.18. It restarts the application and briefly re-enumerates
USB without clearing NVS settings or split bonds.

References:

- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/programming/uf2boot/>
- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/hardware/>
- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/introduction/>
