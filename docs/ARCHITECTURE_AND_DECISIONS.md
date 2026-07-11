# Architecture and decisions

## Objective

Move the ZMK central role from the battery-powered Glove80 left half to a
USB-powered Makerdiary nRF52840 MDK USB Dongle. Both keyboard halves behave as
BLE peripherals while the computer sees a conventional USB keyboard.

## Role allocation

| Device | ZMK role | Host connection | Persistent responsibilities |
| --- | --- | --- | --- |
| MDK USB dongle | Central | USB, optional host BLE | Keymap, split bonds, layers, host profiles, output and lock state |
| Glove80 left | Peripheral | BLE split link | Matrix scanning, ordinary RGB, physical Magic status rendering |
| Glove80 right | Peripheral | BLE split link | Matrix scanning and ordinary RGB |

The dongle has no physical keys. Its mock scanner and 80-position transform let
the central own the complete Glove80 keymap while receiving positions from both
peripherals.

## Decision: pin the MoErgo revision

Both `config/west.yml` and `.github/workflows/build.yml` use:

```text
2f73a230e2fc7b2bd64a9736181e87bf54338131
```

The pinned reusable workflow and source tree use the same Zephyr 3.5-era board
model. Upgrade them together as a separately tested task.

## Decision: custom no-SoftDevice flash map

The inspected dongle runs UF2 Bootloader 0.7.1 without a SoftDevice. The ZMK
application starts at `0x1000`, settings occupy the region below `0xF4000`, and
the UF2 bootloader region remains read-only. This layout is hardware-specific.

## Decision: dongle owns two split peripherals

The central is configured for two BLE split peripherals. After clearing all
stored state, the controlled startup order is:

1. dongle central;
2. left peripheral;
3. right peripheral.

This makes split source 0 the left half and source 1 the right half. Magic status
and Battery Center use that mapping. Changing the pairing order without clearing
all three devices can invalidate it.

## Decision: battery fetching and host proxy

The dongle enables central battery fetching and exposes two proxy Battery
Services to host software such as ZMK Battery Center. The USB-powered dongle's
own Battery Service is disabled so the host sees only the keyboard halves.

The pinned battery getter reports unavailable data unless all configured split
peripherals are connected. As a result, Magic displays six red LEDs for a
battery row it cannot read.

## Decision: global RGB with a dummy dongle device

The exported keymap contains global `&rgb_ug` behaviors. The dongle needs a
local RGB state owner so it can convert relative commands to absolute state and
relay them consistently to both halves.

The shield therefore declares a one-pixel dummy strip on unused P0.13 and a
dummy external-power device on unused P0.14. No LED or power switch is physically
attached to those pads.

Ordinary RGB is enabled on both Glove80 halves. The right half uses MoErgo's
normal peripheral behavior. The left half additionally renders Magic status.

## Decision: patch the pinned RGB implementation in this repository

MoErgo's stock left-board status implementation directly calls central-only
layer, endpoint, HID, BLE, USB, and battery APIs. It therefore cannot link when
the physical left board is built as a peripheral.

Creating a second long-lived ZMK fork was avoided. Instead:

- `zephyr/module.yml` exposes this repository as an extra Zephyr module;
- the root `CMakeLists.txt` applies `patches/moergo-magic-status.patch` before
  ZMK sources compile;
- the patch must match the exact pinned ZMK revision;
- if a GitHub west cache contains an older version of this integration, only
  the three patch-owned RGB source files are restored to pinned `HEAD` before
  the current patch is checked and applied.

Any ZMK revision upgrade must regenerate and review the patch rather than
silently applying it to changed upstream code.

## Decision: pack final Magic LED state into one command

The global RGB behavior already transports two 32-bit parameters. `param1`
remains `RGB_STATUS`; `param2` carries a compact, marked status payload from the
dongle to both peripherals. The left half consumes it, while boards without the
indicator node ignore it.

The payload contains:

- left and right battery display bars, already quantized to 1-6 LEDs;
- the first six active-layer bits;
- Caps Lock, Num Lock, and Scroll Lock bits;
- four two-bit BLE endpoint display states;
- one two-bit USB display state;
- the preferred-output fallback warning;
- a marker bit that distinguishes transported status from local fallback.

This reproduces the stock visible result without changing the split transport
protocol or sending raw central objects to a peripheral.

## Decision: preserve stock status timing and colors

The left half keeps MoErgo's physical pixel mapping and status renderer:

- battery rows use red/yellow/green and 1-6 LEDs;
- active layers use magenta;
- lock and fallback warnings use red;
- endpoint states use lilac, red, dull green, or white;
- the status overlay fades in, remains visible, fades out, and restores ordinary
  RGB after approximately ten seconds.

## Upgrade risks

Treat each of these as a new engineering task:

- changing the MoErgo ZMK or reusable workflow revision;
- changing split connection counts or the pairing order;
- changing battery proxy behavior;
- altering the Magic payload layout or stock pixel mapping;
- changing dongle board definitions, bootloader versions, or flash partitions;
- replacing the canonical keymap in a way that changes global behavior labels.

Each can affect Kconfig dependencies, devicetree bindings, link behavior, flash
addresses, split command compatibility, or stored bonds.
