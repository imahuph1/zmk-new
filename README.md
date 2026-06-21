# zmk-config

ZMK firmware configuration for a Corne, Lily58, Sofle, and Sweep, built via GitHub Actions.

- [Quick start](#quick-start)
- [Repo layout](#repo-layout)
- [Keyboards](#keyboards)
- [Display](#display)
- [RGB underglow](#rgb-underglow)
- [Dongle / peripheral templates](#dongle--peripheral-templates)
- [ZMK Studio](#zmk-studio)
- [Modules](#modules)
- [Useful tips](#useful-tips)
- [Credits](#credits)

----

## Quick start

1. Fork this repository.
2. Edit the keymap for your board in `config/<keyboard>.keymap` (or use the online [keymap-editor] / [ZMK Studio Web](https://zmk.studio/)).
3. Commit and push.
4. On GitHub, go to **Actions → Build → (latest run) → Artifacts** and download the firmware zip for your board.
5. Flash the `.uf2` files to each half:
   - `nice_corne_left_oled_rgb.uf2` / `nice_corne_right_oled_rgb.uf2`
   - `nice_lily58_left_oled.uf2` / `nice_lily58_right_oled.uf2`
   - `nice_sofle_left_rgb_oled.uf2` / `nice_sofle_right_rgb_oled.uf2`
   - `nice_sweep_left.uf2` / `nice_sweep_right.uf2`
   - `nice_settings_reset.uf2` — flash to both halves if they ever desync; see [Useful tips](#useful-tips).
6. Connect each half to the PC and drag-and-drop the corresponding `.uf2` onto the drive that appears after double-tapping reset.

Disable any board you don't need by commenting out its block in [build.yaml](./build.yaml).

## Repo layout

```
zmk-config
├── build.yaml              # GitHub Actions build matrix
├── boards/
│   ├── nice_nano.overlay        # RGB underglow (WS2812 over SPI) for nice!nano v1
│   ├── nice_nano_v2.overlay     # same, for nice!nano v2
│   └── shields/
│       ├── corne/
│       ├── lily58/
│       ├── sofle/
│       └── sweep/
│           ├── Kconfig.shield / Kconfig.defconfig
│           ├── <name>.dtsi              # matrix transform + kscan
│           ├── <name>.zmk.yml           # shield metadata
│           ├── <name>_left.overlay / <name>_right.overlay
│           ├── <name>_left_peripheral.* # optional, unused template
│           └── <name>_dongle_xiao.* / <name>_dongle_pro_micro.*  # optional, unused templates
├── config/
│   ├── west.yml             # ZMK manifest (pinned revision, see comment in the file)
│   ├── config_keymap-drawer.yaml
│   └── <keyboard>.keymap / <keyboard>.conf
├── keymap-drawer/           # auto-generated keymap diagrams (svg + yaml)
└── snippets/
    └── rgb-config/          # shared RGB underglow Kconfig, used by corne/sofle/lily58
```

## Keyboards

| Keyboard | Shields                  | Display       | RGB | Notes |
|----------|---------------------------|---------------|-----|-------|
| Corne    | `corne_left`, `corne_right` | 128x32 OLED | yes | |
| Lily58   | `lily58_left`, `lily58_right` | 128x32 OLED | no  | encoders supported in `lily58.dtsi`, not wired up in the default keymap |
| Sofle    | `sofle_left`, `sofle_right` | 128x32 OLED | yes | encoders supported |
| Sweep    | `sweep_left`, `sweep_right` | none       | no  | 34-key direct-pin board (same PCB as the upstream ZMK `cradio` shield) |

Each keyboard's keymap lives at `config/<keyboard>.keymap`, with diagrams auto-generated to `keymap-drawer/<keyboard>.svg` on every push that touches a `.keymap` file (see [.github/workflows/keymap-drawer.yaml](./.github/workflows/keymap-drawer.yaml)). Sweep doesn't have a diagram yet — its custom physical layout isn't recognized by the keymap-drawer auto-detection.

`settings_reset` is ZMK's built-in shield for clearing a half's Bluetooth bonds — flash it to fix desynced halves (see [Useful tips](#useful-tips)).

## Display

Corne, Lily58, and Sofle each use a standard 128x32 SSD1306 OLED per half, driven by the [zmk-nice-oled] module pulled in via `config/west.yml` and enabled with the `nice_oled` shield in `build.yaml`. Sweep has no display.

## RGB underglow

Corne and Sofle build with WS2812 underglow enabled via the `rgb-config` snippet (`snippets/rgb-config/rgb-config.conf`), which is applied through the `snippet:` field in their `build.yaml` entries. The actual LED strip wiring lives in `boards/nice_nano.overlay` / `boards/nice_nano_v2.overlay`.

Lily58 and Sweep don't enable this snippet, so they build without RGB.

## Dongle / peripheral templates

`boards/shields/{corne,lily58,sofle}/` each include `_dongle_xiao`, `_dongle_pro_micro`, and `_left_peripheral` shield variants, and `config/west.yml` pulls in three more modules for them (`zmk-dongle-display`, `zmk-dongle-display-view`, `zmk-oled-adapter`). None of these are wired up in `build.yaml` right now — they're kept as a starting point if you want to build a dongle setup later, not something this repo currently builds or tests.

## ZMK Studio

ZMK Studio is enabled on every active build (`CONFIG_ZMK_STUDIO=y`, `CONFIG_ZMK_STUDIO_LOCKING=n`, and the `studio-rpc-usb-uart` snippet), so you can remap keys live from [zmk.studio](https://zmk.studio/) without recompiling:
1. Flash the firmware as usual.
2. Connect the central/master half to your PC.
3. Open [zmk.studio](https://zmk.studio/) and connect to your keyboard.

## Modules

Declared in `config/west.yml`:

| Module | Status | Purpose |
|--------|--------|---------|
| [zmk-nice-oled] | active | OLED display widgets, used by Corne/Lily58/Sofle's `nice_oled` shield |
| [zmk-dongle-display] | optional, unused | dongle battery/status display — only needed by the dongle template variants |
| [zmk-dongle-display-view] | optional, unused | nice!view-style version of the above |
| [zmk-oled-adapter] | optional, unused | adapter for 128x32/64/128 OLEDs without modifying shield files |

## Useful tips

- If both halves desync (won't pair), flash `nice_settings_reset.uf2` to **both** halves, then reflash the actual firmware to each.
- To re-flash a half, double-tap its reset button — it should mount as a USB drive — then drag the `.uf2` onto it.

## Credits

This config was originally based on [mctechnology17/zmk-config](https://github.com/mctechnology17/zmk-config). General ZMK config inspiration also drawn from:
- [englmaxi/zmk-config](https://github.com/englmaxi/zmk-config)
- [caksoylar/zmk-config](https://github.com/caksoylar/zmk-config)
- [urob/zmk-config](https://github.com/urob/zmk-config)
- [infused-kim/zmk-config](https://github.com/infused-kim/zmk-config)

Keymap diagrams generated with [keymap-drawer](https://github.com/caksoylar/keymap-drawer).

[keymap-editor]: https://nickcoutsos.github.io/keymap-editor/
[zmk-nice-oled]: https://github.com/mctechnology17/zmk-nice-oled
[zmk-dongle-display]: https://github.com/englmaxi/zmk-dongle-display
[zmk-dongle-display-view]: https://github.com/mctechnology17/zmk-dongle-display-view
[zmk-oled-adapter]: https://github.com/mctechnology17/zmk-oled-adapter
