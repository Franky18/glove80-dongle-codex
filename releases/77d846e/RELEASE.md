# Hardware-validated release: 77d846e

Validation date: 2026-07-11

## Provenance

- Repository: <https://github.com/Franky18/glove80-dongle-codex>
- Pull request: <https://github.com/Franky18/glove80-dongle-codex/pull/1>
- Source commit: `77d846eed7cf9708602428e3a391cf28df01e720`
- Main merge commit: `68378f5fc16eaf375ed44d5cab84ec87c41e9c5c`
- Artifact-producing Actions run:
  <https://github.com/Franky18/glove80-dongle-codex/actions/runs/29128844270>
- Merged artifact name: `firmware`
- GitHub artifact ID: `8241329223`
- Artifact archive digest:
  `sha256:92c2db089b1208a4aaeea527fee3f20e4b512ef175bc4e75d497ac64f45aa885`

## Inputs

- Canonical keymap: `config/my01.keymap`
- Keymap SHA-256:
  `398a93120bb9413fc7a907f9779291c2729d7315a2bf77f54a5df8dfb72b183f`
- MoErgo ZMK revision:
  `2f73a230e2fc7b2bd64a9736181e87bf54338131`
- Dongle: Makerdiary nRF52840 MDK USB Dongle
- Bootloader: UF2 0.7.1, no SoftDevice

## Files

| File | Size in bytes | Purpose |
| --- | ---: | --- |
| `glove80_dongle-central.uf2` | 469504 | USB-powered ZMK central |
| `glove80_lh-peripheral.uf2` | 348672 | Left peripheral, RGB disabled |
| `glove80_rh-peripheral.uf2` | 364032 | Right peripheral |
| `settings_reset-dongle.uf2` | 94720 | Clear dongle state |
| `settings_reset-glove80_lh.uf2` | 125952 | Clear left-half state |
| `settings_reset-glove80_rh.uf2` | 113664 | Clear right-half state |

See `SHA256SUMS` for immutable file hashes.

## Hardware result

The user flashed all three reset images and all three normal images. Both
Glove80 halves sent working key input to the dongle, the dongle sent working USB
keyboard input to the computer, and the configuration was reported stable in
normal use. The firmware was then merged to `main`.

This validation does not claim exhaustive testing of every layer, macro, host
profile, suspend state, or measured battery-life improvement.

## Distribution

The six UF2 binaries are intentionally not committed to Git. Preserve them as
GitHub Release assets and/or a local backup. Actions artifacts are temporary.
