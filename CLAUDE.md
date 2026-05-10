# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A ZMK firmware user-config for a **Choctopus44v2** split keyboard (44 keys + middle encoder) running on **nice_nano_v2** controllers. There is no local toolchain — firmware is built in CI by `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main` (triggered on every push) and the resulting `*.uf2` artifacts are downloaded from the GitHub Actions run and flashed to each half.

ZMK upstream is pinned to **v0.3.0** in `config/west.yml`. Bumping that revision is the only way to pick up newer ZMK behaviors/bindings — keep an eye on it when adding features that depend on recent upstream changes.

The build matrix lives in `build.yaml` (top level): a single `nice_nano_v2` + `choctopus44` entry. The `.zmk/` directory is a local workspace (gitignored) used only if someone wants to run a west build locally; CI does not use it.

## Layout that matters

- `config/choctopus44.keymap` — the **active** keymap. Edits here are what change the firmware.
- `config/choctopus44.json` — ZMK Studio keymap export (kept in sync with the .keymap by Studio). Don't hand-edit unless you know which side is source of truth for the change you're making.
- `config/choctopus44.conf` — Kconfig overrides for this user-config (currently enables EC11 encoder).
- `config/boards/shields/choctopus44/` — the **shield definition** itself, vendored into this repo (matrix transform, kscan GPIO matrix, encoder, vbatt, default keymap). The shield's bundled `choctopus44.keymap` is shadowed by the top-level `config/choctopus44.keymap` at build time; treat the shield dir as hardware description, not behavior.

## Keymap architecture

Six layers, indexed in this order (referenced by `&mo N` / `&thumbmod N`):

0. `default_layer` — Colemak-DH base. Home-row mods on both halves.
1. `lower_layer` — numbers + F-keys.
2. `symbols`
3. `nav` — arrows, word-jump, alt-tab macros, `&bootloader`.
4. `layer_4` — numpad + extra F-keys.
5. `alttab` — held-while-tabbing layer used by the `altctrl`/`alttab` macros.

Custom behaviors defined in the keymap (not stock ZMK names):
- `hml` / `hmr` — left/right home-row mod hold-taps. `hold-trigger-key-positions` is hand-curated to the *opposite* hand's keys plus thumbs, with `hold-trigger-on-release`. If you change the matrix transform or add/remove keys, these position lists must be re-derived or hold-row-mods will misfire.
- `thumbmod` — `&mo` on hold, `&kp` on tap, used on the thumb cluster to combine layer access with a typeable key.
- `skq` — sticky shift with `quick-release` + `ignore-modifiers`.
- `tapcapsl` / `tapcapsr` — tap-dance: 1 tap = sticky shift, 2 taps = `caps_word`.
- Macros `altctrl` / `alttab` hold a mod + `&mo 5` so the alttab layer is live while the user picks a window.

When editing the keymap, remember the matrix transform in `config/boards/shields/choctopus44/choctopus44.dtsi` defines logical key positions (0–43) — the position numbers in `hold-trigger-key-positions` refer to that transform, not the physical wiring.
