# Hardware profile and compatibility

This configuration is tied to one inspected Makerdiary dongle profile. Do not
reuse its flash map for a different nRF52840 board based only on the MCU name.

## Supported profile

The following exact profile is hardware-validated:

| Property | Required value |
| --- | --- |
| Model | Makerdiary nRF52840 MDK USB Dongle |
| Board ID | `nRF52840-MDK-USB-DONGLE` |
| UF2 bootloader | `0.7.1`, dated 2023-07-20 |
| SoftDevice | not present |
| Application start | `0x00001000` |
| UF2 family ID | `0xADA52840` |
| Bootloader start | `0x000F4000` |

Other Makerdiary revisions, other bootloader layouts, dongles with a SoftDevice,
and generic nRF52840 USB dongles are not validated by this repository. Stop and
open an issue before flashing if any identifying value differs.

## Check your dongle before flashing

1. Unplug the dongle.
2. Hold its button while inserting it into USB.
3. Release the button after connection.
4. Open the mounted `UF2BOOT` volume.
5. Read `INFO_UF2.TXT` and compare the Model, Board-ID, Date, and SoftDevice
   fields with the supported profile above.

The inspected device reported:

```text
UF2 Bootloader 0.7.1 lib/nrfx (v2.0.0) lib/tinyusb (0.12.0-145-g9775e7691) lib/uf2 (remotes/origin/configupdate-9-gadbb8c7)
Model: Makerdiary nRF52840 MDK USB Dongle
Board-ID: nRF52840-MDK-USB-DONGLE
Date: Jul 20 2023
SoftDevice: not found
```

A matching MCU name is not enough. A different application or bootloader start
address can make an otherwise valid UF2 overwrite persistent settings or the
bootloader.

## Flash layout

| Region | Start | End | Size | Policy |
| --- | ---: | ---: | ---: | --- |
| MBR | `0x00000000` | `0x00000FFF` | 4 KiB | Read-only |
| ZMK application | `0x00001000` | `0x000D3FFF` | 844 KiB | Firmware |
| ZMK settings/NVS | `0x000D4000` | `0x000F3FFF` | 128 KiB | Persistent state |
| UF2 bootloader | `0x000F4000` | `0x000FFFFF` | 48 KiB | Read-only |

The authoritative repository definition is
`config/boards/shields/glove80_dongle/glove80_dongle_flash.dtsi`.

Normal and settings-reset UF2 files produced by this repository are application
images. They do not intentionally rewrite the UF2 bootloader region.

## Dongle-only logical devices

The MoErgo RGB implementation needs a central RGB state owner even though the
dongle has no Glove80 LED strip. The shield therefore declares:

- a one-pixel dummy WS2812 SPI strip using unused P0.13;
- a dummy external-power device using unused P0.14;
- a mock key scanner with no physical keys;
- the production Glove80 80-position matrix transform.

No LED strip or power switch is physically attached to P0.13 or P0.14. These
nodes maintain central state. Complete Magic state is packed by the dongle and
rendered by the physical left-half LED strip.

## UF2 entry behavior

Factory bootloader path:

1. Unplug the dongle.
2. Hold its button while inserting it.
3. Release after connection; the LED should turn green and `UF2BOOT` should
   mount.

Once the application configures the button as an active-low reset input, a
second path is available: press the button twice within 500 ms. The double-reset
logic already exists in the stock bootloader; ordinary application UF2 files do
not add or replace it.

A single button press while normal firmware is running restarts the application.
It does not erase NVS settings or split bonds. Holding the button while already
connected keeps the MCU in reset; use hold-while-inserting or double-reset for
UF2 mode.

References:

- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/programming/uf2boot/>
- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/hardware/>
- <https://wiki.makerdiary.com/nrf52840-mdk-usb-dongle/introduction/>
