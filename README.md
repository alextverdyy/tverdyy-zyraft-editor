# zyraft — keymap-editor compatible ZMK config

A 34-key split keyboard (Aurora Sweep / Ferris-class) running ZMK firmware, configured to be edited visually in [nickcoutsos/keymap-editor](https://nickcoutsos.github.io/keymap-editor/).

## What's different from the original

This is a **restructured version** of the original `twerd_zyraft` repo (`alextverdyy/twerd_zyraft`, branch `totem-keymap-port`). The original used:

- `#define ZMK_BASE_LAYER(name, ...)` macro indirection
- Numeric layer indices (`DEF=0`, `NAV=1`, ...)
- Custom `base.keymap` shared with other boards in `tverdyy-zmk-config`

This repo instead uses the **keymap-editor-compatible structure**:

- Named layers (`base`, `nav`, `num`, `sym`, `mouse`, `sys`)
- All behaviors inlined in a single `.keymap` file
- `compatible = "..."` declarations on each section
- `dt-formatter` directives for clean round-trip formatting

## The new prototype — what's changed

| Behavior | Old (totem-keymap-port) | New (this repo) |
|---|---|---|
| LH1 (left thumb inner) | `magic_shift` (4-mode: shift / alpha-repeat / sticky / caps-word) | `smart_shift` (hold-tap: tap=sticky shift, hold=held shift) |
| RH0 (right thumb inner) | `&lt FN RET` | `rh0_smart` (mod-morph: RET \| TAB with sticky-shift chord) |
| RH1 (right thumb outer) | `&lt NUM BSPC` | `rh1_smart` (mod-morph: BSPC \| ESC with sticky-shift chord) |
| Tab access | Combo `LM3 LM2` (Alt+Shift+Tab — not even plain Tab!) | Tap LH1 then tap RH0 (smart-shift chord) |
| Esc access | Combo `LT3 LT2` OR cross-thumb `LH0 RH0` | Tap LH1 then tap RH1 (smart-shift chord) |
| tmux prefix | Combo `LB4 LB3` (ZX, collides with normal typing) | Smart-toggle `tmux_prefix` on NAV.RB4 |
| NUM layer | Hold RH1 thumb | Sticky-layer combo on LH0+RH0 cross-thumb |
| SYS layer | Conditional FN+NUM → SYS | 3-thumb combo LH1+LH0+RH0 held |
| FN layer | Separate layer | Merged into SYM (F1–F12 on bottom row) |

### Chord mechanic (the key insight)

The user reported that combos for high-frequency actions (Esc, Tab) felt heavy. The new approach uses **smart-shift chords**:

```
tap LH1 (smart_shift) → sticky-shift pending
tap RH0 → consume shift → emit TAB
tap RH1 → consume shift → emit ESC
```

No holding required. Pure chord.

## Files

```
config/
├── zyraft.keymap     # Single keymap file, keymap-editor compatible
├── zyraft.json       # Keyboard layout for the visual editor
├── west.yml          # ZMK build manifest (modules including prospector)
└── boards/
    └── shields/
        └── zyraft/    # Hardware shield definitions (left/right/dongle)
            ├── zyraft.dtsi, zyraft-layouts.dtsi
            ├── zyraft_left.conf, zyraft_left.overlay
            ├── zyraft_right.conf, zyraft_right.overlay
            ├── zyraft_dongle.conf, zyraft_dongle.overlay
            ├── Kconfig.defconfig, Kconfig.shield
            └── CMakeLists.txt

build.yaml             # GitHub Actions build matrix
.github/workflows/build.yml  # Reusable workflow from zmkfirmware
```

## Editing in keymap-editor

1. Go to https://nickcoutsos.github.io/keymap-editor/
2. Choose "GitHub" as source, select this repo and the `config/zyraft.keymap` file
3. Edit visually. Behaviors from `oskey`, `zmk-helpers`, `zmk-smart-toggle` modules appear as references (not editable but bindable).
4. Commit changes back via the editor.

## Building locally

```bash
cd /path/to/this/repo
# Initialize west workspace
west init -l config
west update
# Build for the dongle (with Prospector screen)
west build -s zmk/app -b xiao_ble -- -DSHIELD="zyraft_dongle prospector_adapter" -DZMK_CONFIG=config
# Or for the left half
west build -s zmk/app -b nice_nano -- -DSHIELD="zyraft_left" -DZMK_CONFIG=config
```

The artifact will be in `build/` after a successful build.

## Modules used (west.yml)

- `zmk` (main)
- `zmk-helpers`, `zmk-unicode`, `zmk-auto-layer`, `zmk-adaptive-key`, `zmk-leader-key` (urob)
- `zmk-tri-state` (dhruvinsh)
- `oskey` (mentaldesk) — for OS-aware key bindings
- `zmk-smart-toggle` (caksoylar) — for the tmux_prefix behavior
- `prospector-zmk-module` (carrefinho) — for the OLED screen on the dongle

## Build artifacts (build.yaml)

| Artifact | Board | Shield | Notes |
|---|---|---|---|
| `settings_reset` | nice_nano | settings_reset | Factory reset (flash before target) |
| `zyraft_dongle` | xiao_ble | zyraft_dongle + prospector_adapter | Central with Prospector screen, ZMK Studio |
| `zyraft_dongle_nostudio` | xiao_ble | zyraft_dongle + prospector_adapter | Central, no ZMK Studio |
| `zyraft_left` | nice_nano | zyraft_left | Left peripheral |
| `zyraft_right` | nice_nano | zyraft_right | Right peripheral |