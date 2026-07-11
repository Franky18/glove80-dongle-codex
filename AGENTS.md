# Repository instructions

These instructions apply to this entire repository.

## Repository boundary

- The only user-owned GitHub repository in scope is
  `https://github.com/Franky18/glove80-dongle-codex.git`.
- Do not enumerate, inspect, clone, modify, or otherwise interact with the
  owner's other repositories.
- Pass `Franky18/glove80-dongle-codex` explicitly to GitHub commands instead of
  relying on ambient repository selection.
- Treat local UF2 files and hardware state as user-owned. Never flash a device
  or erase settings without an explicit request.

## Local workspace layout

This Git repository normally lives at:

```text
glove80dongle/glove80-dongle-zmk-config/
```

Its sibling directories are intentionally outside Git:

- `../firmware/9d42a5f-full-magic-current/`: current hardware-validated UF2 set;
- `../firmware/`: older hardware-test checkpoints;
- `../archives/`: downloaded ZIP archives;
- `../hardware/INFO_UF2.TXT`: inspected dongle bootloader metadata;
- `../legacy/`: preserved pre-sync local files; reference only, not source.

Do not commit UF2 files, downloaded archives, `INFO_UF2.TXT`, or anything from
the sibling `legacy` directory.

## Current hardware-validated baseline

- Main merge commit: `d444c8e1128b26575b770120b68441d5d157b194`.
- Feature source commit: `9d42a5f24de3b4b362d9a06e4f86cdd36f79d113`.
- GitHub Actions run:
  `https://github.com/Franky18/glove80-dongle-codex/actions/runs/29146909546`.
- Hardware validation confirmed typing from both halves, Battery Center values,
  ordinary RGB on both halves, two-row battery status, and the complete stock
  Magic status presentation on the left half.
- The current validated firmware is stored in
  `../firmware/9d42a5f-full-magic-current/`.

Do not describe a later build as hardware-validated until the user confirms it.

## Project invariants

- `config/my01.keymap` is the canonical keymap. Both
  `config/glove80.keymap` and `config/glove80_dongle.keymap` must continue to
  include it.
- The canonical keymap SHA-256 is currently
  `398a93120bb9413fc7a907f9779291c2729d7315a2bf77f54a5df8dfb72b183f`.
- The inspected hardware is the Makerdiary nRF52840 MDK USB Dongle with UF2
  Bootloader 0.7.1, no SoftDevice, and application start address `0x1000`.
- Keep the MoErgo ZMK revision and reusable workflow pinned to
  `2f73a230e2fc7b2bd64a9736181e87bf54338131` unless an upgrade is a deliberate,
  separately tested task.
- `build.yaml` must continue to produce exactly six uniquely named artifacts:
  the dongle central, two Glove80 peripherals, and one settings-reset image for
  each physical device.
- The dongle is the split central. Both Glove80 halves are BLE peripherals.
- Split source 0 is left and source 1 is right after the controlled reset and
  pairing order documented in `docs/FLASHING_AND_RECOVERY.md`.
- Battery fetching and proxy services on the dongle are required for ZMK Battery
  Center and for Magic status battery rows.
- Ordinary RGB is enabled on both halves. The dongle's one-pixel RGB strip and
  external-power nodes are logical state holders only; P0.13 and P0.14 are
  intentionally unused physical pads.
- Complete Magic status support is implemented by
  `patches/moergo-magic-status.patch`, applied through the root Zephyr module.
  The build helper may restore only the three patch-owned ZMK RGB source files
  when upgrading a stale west cache.
- Do not change the flash partition map without re-reading the target dongle's
  `INFO_UF2.TXT` and checking every generated UF2 address range.

## Change and validation workflow

1. Read `docs/PROJECT_STATE.md` and the relevant topic document before editing.
2. Keep changes focused and preserve unrelated user work.
3. Start implementation branches with `agent/` and keep `main` at the last
   hardware-validated state.
4. Validate YAML, the stored ZMK patch, and `git diff --check`.
5. For keymap changes, confirm both wrapper files still include
   `config/my01.keymap` and record the new keymap SHA-256.
6. Build all six matrix entries through GitHub Actions.
7. Require every matrix job and `Merge Output Artifacts` to pass.
8. Confirm the merged artifact contains exactly the six expected UF2 files.
9. Hardware validation is a separate user-performed step.
10. After hardware validation, update `docs/PROJECT_STATE.md` and add or update a
    release manifest under `releases/<source-short-sha>/`.

## Windows migration status

- Windows migration is planned but has not been implemented or validated yet.
- GitHub Actions remains the authoritative build environment and does not
  require a local ZMK toolchain.
- Keep repository paths case-safe, avoid symlinks, preserve LF line endings,
  and prefer scripts that work from PowerShell when Windows support is added.
- Do not claim Windows flashing, local builds, or helper scripts work until they
  are tested on the user's Windows machine.

## Documentation map

- `README.md`: user-facing overview.
- `docs/PROJECT_STATE.md`: current verified state and release pointers.
- `docs/HARDWARE_PROFILE.md`: inspected dongle and flash layout.
- `docs/ARCHITECTURE_AND_DECISIONS.md`: topology and design rationale.
- `docs/BUILD_AND_RELEASE.md`: reproducible build and release procedure.
- `docs/FLASHING_AND_RECOVERY.md`: flashing, pairing, troubleshooting, and
  stock-topology recovery.
- `releases/9d42a5f/`: current hardware-validated release record.
- `releases/77d846e/`: preserved initial dongle-topology validation record.
