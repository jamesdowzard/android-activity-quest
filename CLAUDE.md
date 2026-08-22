# android-activity-quest

Self-owned vendor of `rust-mobile/android-activity` for the Quest 3 / VR-player stack. **Not a fork.** Dissociated 2026-04-26.

## Why this exists

Meta Quest 3 (HorizonOS 12) fires `onNativeWindowResized` / `onNativeWindowRedrawNeeded` outside the documented Android lifecycle. Upstream's unconditional `unwrap()` panics across the FFI boundary and aborts. We need a patched copy and we need to own it independently of upstream so we can evolve it without coordinating PRs back.

## Layout

- `android-activity/` — the actual crate (`name = "android-activity"`, `version = "0.6.1"`).
- `examples/` — minimal upstream examples; kept for smoke tests.

## Constraints

- **Do NOT add `rust-mobile/android-activity` as a git remote.** This repo is dissociated by design. If you want upstream commits, cherry-pick the diffs by hand. Adding a remote and merging would re-entangle history.
- **Do NOT publish to crates.io.** The crate `name = "android-activity"` collides with upstream's published name; consumers use `[patch.crates-io]` against this git repo, not a registry version.
- **Do NOT rename the package** in `android-activity/Cargo.toml`. `[patch.crates-io]` resolves by crate name; renaming breaks every consumer.
- **License files at `android-activity/LICENSE-{MIT,APACHE}` are non-removable.** Upstream contributors' copyright must be preserved.

## Pulling in upstream changes

If a future upstream release is worth a look:

1. Read the upstream diff in a browser; do not clone or add as remote.
2. Apply selected hunks manually here (`/f:new` worktree).
3. Re-verify the Quest patch still applies cleanly to `glue.rs`; re-apply on top if upstream refactored.
4. Bump `version` with `+quest.<n>` suffix (e.g. `0.6.2+quest.1`) to make the divergence explicit.

## Consumers

- `~/code/personal/vr-player/player-rust/Cargo.toml` — `[patch.crates-io]` entry. Update the URL/branch here if this repo moves.

## Provenance

- Base: upstream `0.6.1` at `b4ddf059b77be12cdb955d394e65a92e7568d936`.
- Quest patch: originally `jamesdowzard/android-activity@59a8a6bc` (soft fork, dissociated 2026-04-26).

## Quest device control

Device state, headless verification and logs live in `~/code/personal/quest/`
(CLI `quest …`, MCP `mcp__quest__*`, skill `/quest`). Do not hand-roll `adb`
for these.

- **Never ask James to wear the headset.** `quest device force-worn` disarms the
  proximity sensor so XR apps render at full frame rate with it on a desk.
  Survives reboot.
- **`adb logcat -s` matches tags EXACTLY.** Rust `android_logger` tags by module
  path (`crate::mod::sub`), so exact-match filters hide almost everything and
  mimic a hung app. Use `quest device logcat --prefix <crate>`.
- **Only one immersive app holds the slot.** `am start` succeeding with no
  process forked means another VR app owns it — force-stop it first.
- **Screenshots are 8-bit** — fine for layout and gross brightness, useless for
  banding or bit-depth questions.
- `quest device extensions <pkg>` names the exact manifest string any gated
  OpenXR extension is missing.
