# Build and release

## Authoritative inputs

- User keymap: `config/glove80.keymap`
- Dongle keymap entry point: `config/glove80_dongle.keymap`
- Build matrix: `build.yaml`
- Pinned source manifest: `config/west.yml`
- Dongle shield: `config/boards/shields/glove80_dongle/`
- Magic integration patch: `patches/moergo-magic-status.patch`
- Zephyr module loader: `CMakeLists.txt` and `zephyr/module.yml`
- Reusable workflow: `.github/workflows/build.yml`

The keymap logic runs on the dongle central. Both Glove80 images are split
peripherals, and both retain ordinary RGB support.

The upstream repository ships the official Layout Editor export named
**Glove80 Factory Default Layout** as `config/glove80.keymap`. Forks can build
that default unchanged or replace the file with another Layout Editor export.

## Recommended public build path

GitHub Actions is the supported, platform-independent build environment. A
normal user needs a GitHub fork and a browser; no local ZMK toolchain is
required.

1. Fork `Franky18/glove80-dongle-codex`.
2. Enable workflows from the fork's **Actions** tab if GitHub has disabled them.
3. Replace `config/glove80.keymap` with a Glove80 Layout Editor export.
4. Commit the change to the fork.
5. Open the resulting **Build Glove80 dongle firmware** run.
6. Require `Validate public configuration` and all six firmware build jobs to
   pass.
7. Download the merged artifact named `firmware`.
8. Confirm that it contains exactly:

   ```text
   glove80_dongle-central.uf2
   glove80_lh-peripheral.uf2
   glove80_rh-peripheral.uf2
   settings_reset-dongle.uf2
   settings_reset-glove80_lh.uf2
   settings_reset-glove80_rh.uf2
   ```

9. Record the source commit, Actions run URL, and SHA-256 hashes before flashing.

A green build proves compilation and packaging. It does not prove that a custom
keymap or firmware change works on physical hardware.

## Updating a keymap

1. Export the new layout from the MoErgo Glove80 Layout Editor.
2. Replace `config/glove80.keymap`; keep that filename.
3. Confirm `config/glove80_dongle.keymap` still includes `glove80.keymap`.
4. Review Magic, RGB, output, Bluetooth, bootloader, and layer behaviors because
   those execute on the dongle central.
5. Build all six matrix entries.
6. For an installed dongle topology, follow the routine update procedure and do
   not use settings-reset images.

See [Custom keymaps](CUSTOM_KEYMAP.md) for the feature compatibility contract.

## Validation performed by CI

The validation job checks the public repository contract before invoking the
pinned MoErgo build workflow:

- the canonical keymap and dongle wrapper exist;
- the wrapper includes `glove80.keymap`;
- `build.yaml` contains the six expected unique artifact names;
- the ZMK revision in `config/west.yml` matches the reusable workflow revision;
- generated firmware and binary files are not committed.

The firmware job then builds the matrix defined by `build.yaml` and merges its
outputs into one artifact.

## Local builds

A portable repo-relative local build helper is not currently included. Until one
is added and tested from a clean checkout, GitHub Actions remains the supported
build path for fork users. Do not treat matching UF2 file sizes as proof that
separate build environments produced identical firmware; compare hashes and test
the result.

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

## Hardware validation

For a normal feature update that does not change topology or bonds, flash only
normal firmware images. Do not use settings-reset images unless persistent state
must intentionally be cleared.

At minimum, verify:

- input from both halves through the dongle;
- reconnect after dongle restart;
- Battery Center values for both halves, when host BLE is intentionally used;
- ordinary RGB on both halves;
- Magic battery rows and complete status indicators;
- restoration of the normal RGB effect after the status timeout.

## Project release procedure

1. Build all six images from a reviewable branch or pull request.
2. Record the source commit and Actions run ID.
3. Download and hash the merged artifact.
4. Perform proportional hardware validation.
5. Merge only after the result is reviewed and, where required, hardware-tested.
6. Update `docs/PROJECT_STATE.md`.
7. Add `releases/<source-short-sha>/RELEASE.md` and `SHA256SUMS`.
8. Publish the six UF2 files as GitHub Release assets when durable binary
   distribution is desired.

GitHub Actions artifacts expire. Release records in this repository are
immutable provenance records; do not rewrite an older validated release to
describe a newer build.
