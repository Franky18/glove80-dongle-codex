# Contributing

Thanks for helping make the Glove80 dongle configuration safer and easier to
use. Firmware mistakes can erase settings or make recovery difficult, so issues
and pull requests need enough provenance to reproduce the result.

## Before opening an issue

Read these documents first:

- `README.md` for the supported workflow;
- `docs/HARDWARE_PROFILE.md` for the exact dongle compatibility gate;
- `docs/FLASHING_AND_RECOVERY.md` for reset, pairing, update, and recovery;
- `docs/CUSTOM_KEYMAP.md` for the user-editable keymap contract.

Search existing issues, then collect:

- the complete dongle `INFO_UF2.TXT`;
- repository and source commit;
- GitHub Actions run URL;
- confirmation that all UF2 files came from that one run;
- operating system and Bluetooth adapter;
- exact reset, flash, and startup order;
- whether Windows or another host has a `Glove80 Dongle` BLE bond;
- what worked before the failure and what action restores operation.

Do not post private macro contents or other sensitive text from a personal
keymap. Reduce a problem to a minimal keymap when possible.

## Keymap questions

A personal keymap belongs in the user's fork. Replace only
`config/glove80.keymap` for ordinary customization. Upstream pull requests should
not replace the public example with an unrelated personal layout.

Open an issue when a normal current Layout Editor export fails against the
pinned build without modifying platform files.

## Pull requests

Keep changes focused and explain whether they affect documentation, user
keymaps, build infrastructure, hardware layout, split behavior, or persistent
state.

Before requesting review:

1. confirm `config/glove80_dongle.keymap` still includes `glove80.keymap`;
2. keep `config/west.yml` and the reusable workflow on the same revision;
3. keep the six expected artifact names unique;
4. run `git diff --check`;
5. let all GitHub Actions validation and build jobs pass;
6. record hardware testing separately from compilation.

Do not commit UF2 files, downloaded archives, build directories, west
workspaces, logs containing private data, or hardware metadata copied from a
personal machine. Attach release binaries to a GitHub Release after review and
hardware validation instead.

## High-risk changes

Changes to any of the following require a dedicated proposal and complete
review:

- dongle flash partitions or UF2 family;
- supported board or bootloader profile;
- MoErgo ZMK revision or reusable workflow revision;
- split peripheral count, role allocation, or pairing order;
- battery proxy behavior;
- Magic status payload, RGB patch, or physical pixel mapping.

Do not weaken revision checks or broaden hardware claims solely to make a build
pass.

## Hardware validation

A successful build means that source compiled and artifacts were packaged. Only
describe a build as hardware-validated after someone reports the hardware,
files, source commit, test scope, and result. Accepted validation is recorded
under `docs/PROJECT_STATE.md` and `releases/<source-short-sha>/`.
