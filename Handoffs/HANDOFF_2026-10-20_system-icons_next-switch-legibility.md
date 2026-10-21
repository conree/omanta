# omanta fork — handoff 2026-10-20

Fork: https://github.com/conree/omanta (upstream https://github.com/28allday/omanta)
Clone: `/home/si/Projects/omanta`

## State at end of session

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

## NEXT CHANGE (planned for 2026-10-21): make the on/off state of the Preferences switches readable

### Problem
The user turned the "Use System Icon Theme" switch off by accident. With this theme, an ON switch shows:
- the square grey handle on the RIGHT;
- the left part of the track filled with the purple accent.

The purple filled part reads as the "selected" side, so ON looks like OFF. The user's screenshot showed the switch ON, but the user believed it was off.

### Where
- `qml/PreferencesDialog.qml:169`, `component PrefSwitch: Switch { ... }`. Only padding is customised; the indicator is the stock control style.
- No `QQuickStyle` is set anywhere in `src/` or `qml/`. Qt therefore picks its default Controls style. Fusion is assumed from its look but not confirmed: check with `QQuickStyle::name()` or `QT_QUICK_CONTROLS_STYLE`.
- `PrefSwitch` is used 9 times in that file, so one fix covers every Preferences switch.

### Suggested approach (not started; confirm with the user before building)
1. Give `PrefSwitch` a custom `indicator`:
   - track: filled `Colors.accent` when checked, outline only (`Colors.textDim` border, transparent fill) when unchecked;
   - handle: a round knob in a contrasting colour (`Colors.text` or white), animated from left (off) to right (on).
2. Optionally show a small "On" / "Off" text label next to the switch (`Colors.textDim`).
3. Use theme colours only (`Colors.*`), so it follows `omarchy theme set`. Check a light theme too (Alucard).

### Steps when building
1. Work on a new branch off `system-icon-theme`, or commit straight to it if the user agrees.
2. Run `./bin/test` and confirm all 26 suites pass.
3. Test the change with the user's real settings left untouched, using two variables together:
   - `OMANTA_SETTINGS_FILE=<copy>` so the test uses a copy of the settings file;
   - `dbus-run-session --` so the test doesn't hand off to the running omanta.
4. Screenshot the window with `grim -g` (omanta was on monitor DP-2, workspace 2).
5. Bump `pkgrel` to 3 in `packaging/PKGBUILD`, push, `makepkg` in the scratchpad, and have the user run `sudo pacman -U <pkg>`.
6. Offer it upstream as a SEPARATE PR, branched from `upstream/master`. Leave PR #43 alone.

## Gotchas
- Plain `grep` is a broken shell function on this machine; use `/usr/bin/grep`.
- The running omanta is a single instance. Launching the dev build without `dbus-run-session` just opens a window in the installed copy.
- Git identity for this repo is set locally: user.name `conree`, user.email `sji260@gmail.com`.
- Related system-wide changes made the same day are in `/home/si/Documents/Hand-off_Documents/SYSTEM_CHANGES_HANDOFF_2026-10-20.md`.
