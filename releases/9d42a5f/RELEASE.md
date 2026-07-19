# Hardware-validated release: 9d42a5f

Validation date: 2026-07-11

## Provenance

- Repository: <https://github.com/Franky18/glove80-dongle-codex>
- Pull request: <https://github.com/Franky18/glove80-dongle-codex/pull/5>
- Feature source commit: `9d42a5f24de3b4b362d9a06e4f86cdd36f79d113`
- Main merge commit: `d444c8e1128b26575b770120b68441d5d157b194`
- Artifact-producing Actions run:
  <https://github.com/Franky18/glove80-dongle-codex/actions/runs/29146909546>
- Merged artifact name: `firmware`
- GitHub artifact ID: `8247032286`
- Artifact archive digest:
  `sha256:2a35f05eeed574553474d55ed815443676688496b15a8fe034eab62e0af3b473`

## Inputs

- Keymap: layout recorded by the SHA-256 value below
- Keymap SHA-256:
  `398a93120bb9413fc7a907f9779291c2729d7315a2bf77f54a5df8dfb72b183f`
- MoErgo ZMK revision:
  `2f73a230e2fc7b2bd64a9736181e87bf54338131`
- Dongle: Makerdiary nRF52840 MDK USB Dongle
- Bootloader: UF2 0.7.1, no SoftDevice

## Files

| File | Size in bytes | Purpose |
| --- | ---: | --- |
| `glove80_dongle-central.uf2` | 472576 | USB/BLE split central and complete Magic state owner |
| `glove80_lh-peripheral.uf2` | 365056 | Left peripheral, ordinary RGB and Magic renderer |
| `glove80_rh-peripheral.uf2` | 364032 | Right peripheral with ordinary RGB |
| `settings_reset-dongle.uf2` | 94720 | Clear dongle settings and bonds |
| `settings_reset-glove80_lh.uf2` | 126464 | Clear left-half settings and bond |
| `settings_reset-glove80_rh.uf2` | 113664 | Clear right-half settings and bond |

See `SHA256SUMS` for immutable file hashes.

## Hardware result

The user confirmed:

- input from both halves through the dongle;
- correct left and right values in ZMK Battery Center;
- ordinary RGB on both halves;
- stock-style two-row battery status after a Magic tap;
- the complete Magic status presentation for layers, locks, BLE, USB, and
  output fallback;
- automatic restoration of the normal RGB effect after the status timeout.

The implementation was then merged to `main`.

## Distribution

The six UF2 binaries are intentionally not committed to Git. Publish them as
GitHub Release assets for durable distribution; Actions artifacts are temporary.
