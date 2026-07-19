## Summary

Describe the user-visible change and why it is needed.

## Scope

- [ ] Documentation or onboarding
- [ ] Keymap compatibility
- [ ] Build or release workflow
- [ ] Connectivity, pairing, or persistent state
- [ ] Battery, Magic status, or RGB
- [ ] Board, bootloader, or flash layout

## Validation

- [ ] `config/glove80_dongle.keymap` still includes `glove80.keymap`.
- [ ] The ZMK manifest pin matches the reusable workflow pin.
- [ ] The build matrix still produces exactly six unique expected artifacts.
- [ ] `git diff --check` passes.
- [ ] `Validate public configuration` passes.
- [ ] All six firmware jobs and artifact merge pass.
- [ ] No UF2 files, archives, private keymap data, or local hardware metadata are committed.

## Hardware result

State **not hardware-tested**, or record the exact hardware profile, source
commit, firmware hashes, test scope, and result. A successful build alone is not
hardware validation.
