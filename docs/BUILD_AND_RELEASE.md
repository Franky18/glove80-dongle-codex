# Build and release

## Authoritative inputs

- Canonical keymap: `config/my01.keymap`
- Glove80 entry point: `config/glove80.keymap`
- Dongle entry point: `config/glove80_dongle.keymap`
- Build matrix: `build.yaml`
- Pinned source manifest: `config/west.yml`
- Dongle shield: `config/boards/shields/glove80_dongle/`
- Magic integration patch: `patches/moergo-magic-status.patch`
- Zephyr module loader: `CMakeLists.txt` and `zephyr/module.yml`
- Reusable workflow: `.github/workflows/build.yml`

The keymap logic runs on the dongle central. Both Glove80 images are split
peripherals, and both retain ordinary RGB support.

## Recommended build path

GitHub Actions is the authoritative and platform-independent build environment.
macOS or Windows only needs Git/GitHub access and a browser to trigger a build
and retrieve artifacts; a local ZMK toolchain is not required for the normal
workflow.

1. Push a focused branch to
   `Franky18/glove80-dongle-codex`.
2. Open a draft pull request against `main`.
3. Wait for all six build jobs and `Merge Output Artifacts`.
4. Download the merged artifact named `firmware`.
5. Confirm that it contains exactly:

   ```text
   glove80_dongle-central.uf2
   glove80_lh-peripheral.uf2
   glove80_rh-peripheral.uf2
   settings_reset-dongle.uf2
   settings_reset-glove80_lh.uf2
   settings_reset-glove80_rh.uf2
   ```

6. Record SHA-256 hashes before hardware testing.

A green build proves compilation and packaging, not hardware behavior.

## Updating the keymap

1. Export a keymap from the MoErgo Layout Editor.
2. Replace `config/my01.keymap`; do not create a second canonical copy.
3. Confirm both wrapper files still include `my01.keymap`.
4. Record its SHA-256.
5. Review changes to Magic, RGB, output, Bluetooth, and layer behavior because
   those execute on the dongle central.

The current keymap SHA-256 is:

```text
398a93120bb9413fc7a907f9779291c2729d7315a2bf77f54a5df8dfb72b183f
```

## Updating the ZMK integration patch

`patches/moergo-magic-status.patch` is generated against the exact revision in
`config/west.yml` and changes three upstream files:

- `app/include/zmk/rgb_underglow.h`;
- `app/src/behaviors/behavior_rgb_underglow.c`;
- `app/src/rgb_underglow.c`.

Before pushing a patch change:

1. verify it applies cleanly to a pristine checkout of the pinned revision;
2. verify a second configuration pass recognizes it as already applied;
3. verify the cache-upgrade path can recover from the previous stored patch;
4. review the complete applied ZMK diff;
5. build all six matrix entries.

Do not weaken the fail-fast revision check merely to make a new ZMK revision
compile.

## Structural checks

On macOS/Linux:

```sh
ruby -e 'require "yaml"; d=YAML.safe_load(File.read("build.yaml")); raise unless d["include"].length == 6'
rg -n '^#include "my01.keymap"' config/glove80.keymap config/glove80_dongle.keymap
shasum -a 256 config/my01.keymap
git diff --check
```

Equivalent PowerShell checks will be designed and tested during the Windows
migration. Until then, do not present untested Windows commands as validated.

## Hardware validation

For a normal feature update that does not change topology or bonds, flash only
the normal images affected by the change. Do not use settings-reset images
unless persistent state must intentionally be cleared.

At minimum, verify:

- input from both halves through the dongle;
- reconnect after dongle restart;
- Battery Center values for both halves;
- ordinary RGB on both halves;
- Magic battery rows and complete status indicators;
- restoration of the normal RGB effect after the status timeout.

## Release procedure

1. Build all six images from a reviewable branch or pull request.
2. Record the source commit and Actions run ID.
3. Download and hash the merged artifact.
4. Perform proportional hardware validation.
5. Merge only after the user confirms the result when practical.
6. Update `docs/PROJECT_STATE.md`.
7. Add `releases/<source-short-sha>/RELEASE.md` and `SHA256SUMS`.
8. Preserve the six UF2 files outside Git and optionally attach them to a GitHub
   Release for durable distribution.

GitHub Actions artifacts expire. Local UF2 backups are stored in the workspace
`firmware/` directory; generated binaries remain excluded from Git.
