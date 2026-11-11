# Corne ZMK Config

Personal ZMK firmware for a **42-key split Corne keyboard** running on Nice!Nano v2.

## Hardware

| Part | Detail |
|---|---|
| Builder | Saurav ([PARIX](https://parix.in) / [@dieselsaurav](https://github.com/dieselsaurav)) |
| Board | Nice!Nano v2 (nRF52840) |
| Shield | Corne (split, 6×3+3 = 42 keys) |
| Display | OLED SSD1306 128×32 |
| RGB | 27 WS2812 LEDs (per-key + underglow) |
| Bluetooth | 4 profiles (BT_SEL 0–3) |
| ZMK Studio | Enabled — remap live at [zmk.studio](https://zmk.studio) |
| Mouse | Enabled — MOUSE layer (hold `Tab`) |
| Deep Sleep | 15 min idle → ~20µA draw |

## Keymap — Miryoku-style (7 layers)

| Layer | Activate | Contents |
|---|---|---|
| 0 BASE | always | QWERTY + GACS home-row mods |
| 1 NAV | hold `Space` | vim arrows (HJKL), clipboard (Ctrl), navigation |
| 2 NUM | hold `Bspc` | numpad on left hand |
| 3 MEDIA | hold `Esc` | media keys, BT profiles, RGB control |
| 4 SYM | hold `Enter` | symbols on left hand |
| 5 FUN | hold `Del` | F1–F12 function keys |
| 6 MOUSE | hold `Tab` | mouse movement, scroll, clicks |

### Home-row Mods (BASE layer)

```
Pinky  → GUI   (A / ')
Ring   → ALT   (S / L)
Middle → CTRL  (D / K)
Index  → SHIFT (F / J)
```

### Key Combos

| Keys | Output |
|---|---|
| TAB + Q | **Bootloader** (left half) |
| P + BSPC | **Bootloader** (right half) |
| F + J | Caps Word |
| W + E | Escape |
| I + O | Backspace |
| O + P | Delete |
| S + D | Tab |
| K + L | Enter |
| X + C | Ctrl+C |
| C + V | Ctrl+V |
| X + V | Ctrl+X |

## Build & Flash

Push to `config/` or `build.yaml` triggers GitHub Actions automatically.

1. Go to [Actions](https://github.com/Rvkt/zmk-config/actions) → latest green run → **Artifacts**
2. Download `firmware.zip` and extract
3. **First time:** flash `settings_reset-nice_nano-zmk.uf2` to both halves first
4. Double-tap reset on left half → `NICENANO` drive appears → drag `corne_left-nice_nano-zmk.uf2`
5. Same for right half with `corne_right-nice_nano-zmk.uf2`

> Always flash **left half first** (it's the central/host).

## ZMK Studio

With keyboard plugged in via USB, visit [zmk.studio](https://zmk.studio) to remap keys live without reflashing.

## Credits & Acknowledgements

- **Hardware Builder & Base Firmware:** Built and assembled by **Saurav** ([@dieselsaurav](https://github.com/dieselsaurav) / [PARIX](https://parix.in)). Initial firmware configuration referenced from [dieselsaurav/zmk-for-keyboards](https://github.com/dieselsaurav/zmk-for-keyboards).
- **Corne Keyboard (CRKBD):** Designed by [Kosuke Adachi (@foostan)](https://github.com/foostan/crkbd).
- **Layout Architecture:** Miryoku ergonomic layout principles by [Manna-Harbour](https://github.com/manna-harbour/miryoku).
- **Firmware Engine:** Powered by the open-source [ZMK Firmware](https://zmk.dev/) project.

