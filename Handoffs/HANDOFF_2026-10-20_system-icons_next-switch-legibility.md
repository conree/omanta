# omanta fork — handoff 2026-10-20

Fork: https://github.com/conree/omanta (upstream https://github.com/28allday/omanta)
Clone: `/home/si/Projects/omanta`

## State at end of session

### Update 2026-10-21 — Preferences controls readability (local source change)
- `qml/PreferencesDialog.qml` now gives the shared `PrefSwitch` an explicit `ON`/`OFF` label, a round handle that moves left/right with the state, and a theme-coloured track when enabled. This addresses the mistaken reading described below for all Preferences switches.
- The two range sliders now show the current value above the track and clear endpoint guidance below it. Reorder Hold Time also labels all four snap values (0.5, 1.0, 1.5, 2.0 seconds); Background Opacity names the transparent and opaque ends.
- Follow-up correction from the user: the switch thumb is now a rectangular segment that is purple on the right when ON and gray on the left when OFF; the opposite side of the track shows the matching `ON`/`OFF` text.
- The Preferences window background and its cards are now fully opaque, retaining their theme RGB colours, so the existing background-opacity preference does not reduce contrast in Preferences.
- User installed the local package `omanta 0.1.29-3` and confirmed that the switch now reads correctly.
- Submitted upstream as [PR #44](https://github.com/28allday/omanta/pull/44), branch `pr/preferences-control-readability` from `upstream/master`. It contains only the Preferences controls and opacity changes; the fork-only system icon preference is excluded. Existing PR #43 remains open and unchanged.
- Debug and package Release builds completed; `qmllint qml/PreferencesDialog.qml` and `git diff --check` pass. Automated tests were not run. The update in the `system-icon-theme` worktree remains uncommitted; the separate PR branch is committed and pushed.
- The workspace instruction prohibits running tests unless requested, so continue with visual review/build packaging only as authorized; do not run `./bin/test` without a direct request.

### Done and installed
- **Branch `system-icon-theme`** (the branch the installed package is built from)
  - `4ea6fbd` adds a "Use System Icon Theme" preference: Preferences → Appearance, setting key `useSystemIcons`, default off.
  - `2da35e1` points `packaging/PKGBUILD` at this fork and branch, as version 0.1.29-2.
- **Installed:** omanta 0.1.29-2 via `sudo pacman -U`. omanta isn't in any pacman repo and Omarchy doesn't manage it, so system updates won't overwrite it.
- **User setting:** `useSystemIcons=true` in `~/.config/omanta/settings`. Folders and files show the Tela-circle-dracula-dark-van-helsing icons. The user confirmed this on screen.
- **Tests:** all 26 suites passed (`./bin/test`). A separate agent reviewed the code and found no bugs.
- **Fonts:** omanta sets no font family of its own (only `monospace` in two dialogs), so it already uses the system font, Visby CF Medium 14.

### Upstream PR
- https://github.com/28allday/omanta/pull/43 is OPEN. Branch `pr/system-icon-theme`, commit `957f1c7`.
  - It contains only the feature commit; the PKGBUILD commit is not included.
- The owner usually closes PRs and folds them into the next squashed release, crediting the contributor in the release notes (see #37 and #39). If a release ships `useSystemIcons`, switch back to upstream builds.

## Change context: make the on/off state of the Preferences switches readable

### Problem
The user turned the "Use System Icon Theme" switch off by accident. With this theme, an ON switch shows:
- the square grey handle on the RIGHT;
- the left part of the track filled with the purple accent.

The purple filled part reads as the "selected" side, so ON looks like OFF. The user's screenshot showed the switch ON, but the user believed it was off.

### Where
- `qml/PreferencesDialog.qml:169`, `component PrefSwitch: Switch { ... }`. The shared component now supplies a custom indicator.
- No `QQuickStyle` is set anywhere in `src/` or `qml/`. Qt therefore picks its default Controls style. Fusion is assumed from its look but not confirmed: check with `QQuickStyle::name()` or `QT_QUICK_CONTROLS_STYLE`.
- `PrefSwitch` is used 9 times in that file, so one fix covers every Preferences switch.

### Implemented locally on 2026-10-21
1. The custom indicator has a gray track and a moving rectangular thumb: `Colors.accent` on the right when checked, `Colors.textDim` on the left when unchecked.
2. It prints `ON` or `OFF` opposite the thumb and uses theme colours (`Colors.*`) so it follows `omarchy theme set`. Light-theme appearance still needs a visual check.

### Remaining release steps (not yet done)
1. Confirm the user's preferred branch/commit workflow before committing; the current edits are uncommitted on `system-icon-theme`.
2. If the user asks for tests, use the isolated settings and D-Bus setup below:
   - `OMANTA_SETTINGS_FILE=<copy>` so the test uses a copy of the settings file;
   - `dbus-run-session --` so the test doesn't hand off to the running omanta.
3. Visually inspect Preferences in both dark and light themes; omanta was on monitor DP-2, workspace 2.
4. If packaging is requested, bump `pkgrel` to 3 in `packaging/PKGBUILD`, build the package, and let the user install it.
5. Offer the change upstream as a SEPARATE PR, branched from `upstream/master`. Leave PR #43 alone.

## Gotchas
- Plain `grep` is a broken shell function on this machine; use `/usr/bin/grep`.
- The running omanta is a single instance. Launching the dev build without `dbus-run-session` just opens a window in the installed copy.
- Git identity for this repo is set locally: user.name `conree`, user.email `sji260@gmail.com`.
- Related system-wide changes made the same day are in `/home/si/Documents/Hand-off_Documents/SYSTEM_CHANGES_HANDOFF_2026-10-20.md`.
