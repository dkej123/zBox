# Publication status

Last updated: 2026-08-31

This document records the current state of the repository-publication work. All changes
described here exist only on the local `public-ready` branch. The working `main` branch and
its staged, unstaged, untracked, and submodule firmware changes have not been modified.

## Completed

- Replaced the root README with a concise English project overview and end-to-end quick start.
- Added a reproducible build guide, current-schematic BOM, and hardware revision status.
- Documented the difference between the manufactured legacy PCB/Gerbers v2.0, the current
  schematic, current firmware expectations, and the unimplemented power-latch proposal.
- Updated firmware, server, hardware, and test documentation to match current button,
  provisioning, sync, sleep, and wake behavior.
- Removed obsolete debugging plans, completed checklists, generic agent notes, and the binary
  embedded-refactoring skill.
- Removed the obsolete Docker Compose `version` field and expanded `.gitignore` for runtime,
  build, credential, backup, and local-agent files.
- Preserved the MIT license, printable card files, CAD, STL, PCB, schematic, and legacy Gerbers.
- Updated the GitHub repository description and topics. No release was created.

## History and identity

- The complete history reachable from `public-ready` was rewritten so the former work address
  is absent from both author and committer metadata.
- Rewritten commits use the authenticated GitHub noreply identity where that work address had
  previously appeared. New publication commits also use the noreply identity.
- The public GitHub `main` branch has not been replaced yet, so its old history remains visible
  until a separately approved history replacement is performed.
- A local recovery bundle of the pre-rewrite publication branch was created outside the
  repository. It must not be published because it intentionally contains the old history.

## Submodule decision

The repository previously pinned `esp32/lib/ESP32-A2DP` to a locally created commit that was
not available from the configured public upstream. On `public-ready`, the gitlink now points to
its publicly available parent, `4783cf6dfe107d00873f5883d3350de1981f253b`. This makes a clean
recursive checkout possible without creating or maintaining a separate fork. Local submodule
work on `main` was not removed or changed.

## Validation completed

- Root README length: 134 lines.
- All retained project Markdown links resolve locally.
- `docker compose config`, image build, persistence mounts, and `/api/health` smoke test pass.
- Python compilation and container import checks pass.
- PlatformIO native tests pass: 132 `native` plus 28 `native_btndec`, 160 total.
- Release firmware build `lolin_d32_pro` passes.
- Debug firmware build `lolin_d32_pro_debug` passes.
- A clean recursive checkout successfully fetches the public ESP32-A2DP revision.
- The BOM was cross-checked against current KiCad references and footprints.
- The history reachable from `public-ready` contains no former work-address metadata.
- The original `main` working tree was checked after the work and remains unchanged.

## Remaining publication steps

1. Review the dependency behavior after dropping the private ESP32-A2DP patch, especially
   Bluetooth AVRCP volume notifications, on physical hardware.
2. Decide when to publish `public-ready` and when to replace public `main` with its rewritten
   history. Replacing `main` will require a coordinated force-push and fresh clones or hard
   resets for any existing collaborators.
3. Repeat the secret/history scan against the exact refs intended for publication immediately
   before pushing.
4. Do not create a v1.0 release until a synchronized successor PCB has been manufactured and
   physically verified.
