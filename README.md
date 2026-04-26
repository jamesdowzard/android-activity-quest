# `android-activity-quest`

Self-owned vendor of [`android-activity`](https://github.com/rust-mobile/android-activity) for the Quest 3 / VR-player stack.

This is **not a fork** in the GitHub sense. It was dissociated from `rust-mobile/android-activity` on 2026-04-26 to be an independent, self-contained dependency that can be evolved without coordinating with upstream.

## What's different from upstream

A single targeted fix on top of upstream `0.6.1`: `WaitableNativeActivityState::notify_window_resized` and `notify_window_redraw_needed` no longer abort the process when called outside the documented `onNativeWindowCreated`/`onNativeWindowDestroyed` lifecycle. Meta Quest 3 (HorizonOS 12) fires these callbacks outside that lifecycle, and the upstream `unwrap()` panics across the FFI boundary into `panic_cannot_unwind` → `process::abort` → `SIGABRT`. The patched callbacks log a warning and ignore the event when the window pointer is missing or mismatched, allowing the app to proceed to `xrCreateInstance`.

See `android-activity/src/native_activity/glue.rs` around `notify_window_resized` and `notify_window_redraw_needed` for the diff.

## Provenance

- Upstream base: `rust-mobile/android-activity@b4ddf059b77be12cdb955d394e65a92e7568d936` ("Release 0.6.1 (take 2)", 2026-03-24, Robert Bragg).
- Patch authored on 2026-04-21 in `vr-player#33` and originally lived on `jamesdowzard/android-activity@59a8a6bc` (a soft fork, now dissociated).
- This repo's history starts at the dissociation commit; the upstream commit history is no longer present locally. Use `rust-mobile/android-activity` directly if you need that history.

## Use as a Cargo dependency

```toml
[patch.crates-io]
android-activity = { git = "https://github.com/jamesdowzard/android-activity-quest", branch = "main" }
```

The crate name remains `android-activity` so `[patch.crates-io]` works without renames.

## Updating from upstream

If a future upstream release is worth pulling in:

1. Cherry-pick the relevant upstream commits into this tree manually (do not add `rust-mobile/android-activity` as a remote — keeps full dissociation).
2. Re-apply the Quest fix on top if upstream regresses or refactors `glue.rs`.
3. Bump `android-activity/Cargo.toml` version with a `+quest.<n>` suffix to make the divergence explicit.

## License

Inherits MIT OR Apache-2.0 from upstream `rust-mobile/android-activity`. License texts preserved at `android-activity/LICENSE-MIT` and `android-activity/LICENSE-APACHE`. Original copyright holders retain their notices.
