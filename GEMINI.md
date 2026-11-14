# GEMINI.md — ZMK Firmware Repository Manual & Architecture Reference

> **Role & Scope:** This document is the authoritative engineering manual and operational specification for AI agents (Gemini / Antigravity) and human firmware engineers maintaining the `zmk-config` repository. It provides complete, verified context on the hardware architecture, Devicetree keymap configuration, Kconfig settings, build pipelines, flashing procedures, and debugging workflows.

---

## 1. Project Overview

### 1.1 Project Purpose
This repository maintains personal [ZMK Firmware](https://zmk.dev/) configuration for a **42-key split Corne (CRKBD)** ergonomic keyboard powered by **nice!nano v2** microcontrollers. The firmware is optimized for a Miryoku-inspired 7-layer layout, Home-Row Mod-Tap behaviors (GACS arrangement), low-latency Bluetooth Low Energy (BLE) split communications, OLED status displays, WS2812 RGB underglow, mouse emulation, and live keymap remapping via ZMK Studio.

### 1.2 Hardware Architecture

| Component | Specification / Value | Status |
|---|---|---|
| **Keyboard Shield** | Corne (CRKBD) split mechanical keyboard | CONFIRMED |
| **Physical Layout** | 42 keys: 3×6 column-staggered matrix + 3 thumb keys per half | CONFIRMED |
| **Hardware Builder** | Saurav ([@dieselsaurav](https://github.com/dieselsaurav) / [PARIX](https://parix.in)); base firmware from [`dieselsaurav/zmk-for-keyboards`](https://github.com/dieselsaurav/zmk-for-keyboards) | CONFIRMED |
| **Microcontrollers** | nice!nano v2 (Nordic Semiconductor nRF52840 SoC, 32-bit ARM Cortex-M4F @ 64 MHz, 1 MB Flash, 256 KB RAM) | CONFIRMED |
| **Split Topology** | Central (Left half) + Peripheral (Right half) | CONFIRMED |
| **Displays** | Dual SSD1306 128×32 I2C OLED displays (one per half) | CONFIRMED |
| **Lighting** | 27 WS2812 addressable RGB LEDs (per-key + underglow) | CONFIRMED |
| **Power Source** | Single-cell Lithium-Polymer (LiPo) / Li-ion 3.7V rechargeable battery per half | CONFIRMED |
| **Battery Capacity** | Standard 110mAh–301230 LiPo cells | NOT VERIFIED (Hardware dependent) |
| **PCB Revision** | Corne v3 / standard 42-key pinout | NOT VERIFIED (Upstream Corne shield standard) |
| **Host OS** | Linux / Omarchy (evdev, BlueZ, X11/Wayland input stack) | CONFIRMED |
| **Connectivity** | Bluetooth Low Energy (BLE 5.0) + USB 2.0 Full-Speed (Type-C) | CONFIRMED |

### 1.3 Firmware Architecture
- **Operating System Core:** Zephyr RTOS (v4.1.0+ compatible via ZMK main branch).
- **Configuration Subsystems:**
  - **Devicetree Source (`.keymap`):** Defines physical key bindings, layer switching, hold-tap behaviors (`&mt`, `&lt`), key combos, and macros.
  - **Kconfig (`.conf`):** Configures Zephyr kernel drivers, BLE stack parameters, OLED display widgets, pointing devices, power management timeouts, and ZMK Studio RPC.
  - **West Manifest (`west.yml`):** Directs workspace initialization by pinning the `zmkfirmware/zmk` upstream project repository and importing `app/west.yml`.
  - **CI Workflow & Build Matrix (`build.yaml` & `build.yml`):** Directs automated cross-compilation across Zephyr targets using `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`.

### 1.4 Repository Structure

```
.
├── .github/
│   └── workflows/
│       └── build.yml          # GitHub Actions workflow (dispatches to upstream build-user-config)
├── config/
│   ├── corne.conf             # Hardware, BLE, OLED, RGB, sleep, and pointing Kconfig flags
│   ├── corne.keymap           # Devicetree 7-layer keymap, behaviors, combos, and macros
│   └── west.yml               # West workspace manifest pointing to zmkfirmware/zmk:main
├── .gitignore                 # Ignores GIT_IGNORE/ local directory
├── build.yaml                 # Build matrix targets (left, right, settings_reset)
├── GEMINI.md                  # Authoritative AI & firmware engineer documentation (this file)
└── README.md                  # User-facing summary of layers, combos, and flashing instructions
```

---

## 2. Development Environment

### 2.1 Required Tools and Dependencies
1. **Cloud CI (Primary):** GitHub Actions runner using `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`.
2. **Local Build Containers (Alternative):** Docker / Podman running `zmkfirmware/zmk-build-arm:stable` or `zmkfirmware/zmk-build-arm:main`.
3. **Local Toolchain (Advanced Native):**
   - Python 3.10+
   - `west` (Zephyr meta-tool: `pip install west`)
   - CMake 3.20.5+
   - Ninja build system
   - Zephyr ARM SDK (`arm-zephyr-eabi` cross-compiler)

### 2.2 Board Target & Qualifiers (Zephyr 4.1.0+ Requirement)
> [!IMPORTANT]
> ZMK upgraded to Zephyr 4.1.0+, requiring board variant qualifiers.
> The confirmed, official board identifier for nice!nano v2 is **`nice_nano//zmk`**.
>
> - **DO NOT USE:** `nice_nano_v2` (deprecated syntax; fails board lookup).
> - **DO NOT USE:** `nice_nano` (lacks the required `//zmk` variant qualifier).
> - **DO NOT USE:** `nice_nano_v2//zmk` (invalid board directory in upstream ZMK).

### 2.3 Build Matrix Specification (`build.yaml`)
The repository specifies three build artifacts in [build.yaml](file:///home/rvkt/zmk-config/build.yaml):

```yaml
---
include:
  # 1. Left Half (Central / Host Controller) + ZMK Studio RPC + vertical OLED (zmk-nice-oled)
  - board: nice_nano_v2
    shield: corne_left nice_oled
    snippet: studio-rpc-usb-uart
    cmake-args: -DCONFIG_ZMK_STUDIO=y
    artifact-name: corne_left-nice_nano-zmk

  # 2. Right Half (Peripheral) + vertical OLED (zmk-nice-oled)
  - board: nice_nano_v2
    shield: corne_right nice_oled
    artifact-name: corne_right-nice_nano-zmk

  # 3. Settings Reset Shield (Emergency BLE Bond & NVS Clear)
  - board: nice_nano_v2
    shield: settings_reset
    artifact-name: settings_reset-nice_nano-zmk
```

### 2.4 Compilation Commands

#### A. Automated CI Build (Recommended)
Triggers automatically on any commit touching `config/**`, `build.yaml`, or `.github/workflows/**`:
```bash
git add config/ build.yaml
git commit -m "feat(keymap): update layer bindings"
git push origin main
```
Alternatively, trigger manually via GitHub Web UI (`Actions` → `Build ZMK Firmware` → `Run workflow`) or GitHub CLI:
```bash
gh workflow run build.yml
```

#### B. Local Native West Build (If Zephyr SDK is installed locally)
```bash
# Initialize west workspace (one-time setup in an empty parent directory)
west init -l config
west update
west zephyr-export

# Build Left Half with ZMK Studio snippet:
west build -s zmk/app -d build/left -b nice_nano//zmk -S studio-rpc-usb-uart -- \
  -DSHIELD=corne_left \
  -DZMK_CONFIG="$(pwd)/config" \
  -DCONFIG_ZMK_STUDIO=y

# Build Right Half:
west build -s zmk/app -d build/right -b nice_nano//zmk -- \
  -DSHIELD=corne_right \
  -DZMK_CONFIG="$(pwd)/config"

# Build Settings Reset UF2:
west build -s zmk/app -d build/reset -b nice_nano//zmk -- \
  -DSHIELD=settings_reset \
  -DZMK_CONFIG="$(pwd)/config"
```

#### C. Local Docker Container Build
```bash
# From repository root
docker run --rm -v "$(pwd):/workspace" -w /workspace \
  zmkfirmware/zmk-build-arm:stable \
  bash -c "
    west init -l config
    west update
    west build -s /zmk/app -d /workspace/build/left -b nice_nano//zmk -S studio-rpc-usb-uart -- -DSHIELD=corne_left -DZMK_CONFIG=/workspace/config -DCONFIG_ZMK_STUDIO=y
    west build -s /zmk/app -d /workspace/build/right -b nice_nano//zmk -- -DSHIELD=corne_right -DZMK_CONFIG=/workspace/config
  "
```

### 2.5 Flashing Instructions
Each half of the keyboard runs an independent UF2 bootloader embedded in the nice!nano v2.

1. **Enter Bootloader Mode:**
   - **Method A (Key Combo):**
     - Left Half: Press `TAB` + `Q` (Positions 0 + 1)
     - Right Half: Press `P` + `BSPC` (Positions 10 + 11)
   - **Method B (Physical Reset Button):** Double-tap the physical reset button on the nice!nano.
2. **Mount Storage Volume:**
   - A USB Mass Storage volume labeled `NICENANO` will mount on Linux (e.g. `/run/media/$USER/NICENANO`).
3. **Transfer Firmware:**
   - Copy `corne_left-nice_nano-zmk.uf2` to the Left half `NICENANO` drive.
   - The microcontroller automatically flashes the ROM, resets, and unmounts the volume.
   - Connect the Right half via USB, enter bootloader mode, and copy `corne_right-nice_nano-zmk.uf2`.
4. **Flashing Order Rule:**
   > [!IMPORTANT]
   > Always flash the **LEFT HALF FIRST**. The left half acts as the Bluetooth central. If the peripheral (right half) is flashed first or disconnected, timing synchronization during BLE pairing can stall.

### 2.6 Linux / Omarchy Development Considerations
- **USB Device Permissions:** Linux systems may require udev rules for WebUSB / WebSerial / ZMK Studio communication:
  `/etc/udev/rules.d/99-zmk.rules`:
  ```udev
  # nice!nano v2 / ZMK Studio USB-UART CDC ACM
  KERNEL=="ttyACM*", ATTRS{idVendor}=="1d50", ATTRS{idProduct}=="615e", MODE="0666", GROUP="uucp"
  SUBSYSTEM=="usb", ATTR{idVendor}=="1d50", ATTR{idProduct}=="615e", MODE="0666", GROUP="uucp"
  ```
- **Bluetooth Administration:**
  - Use `bluetoothctl` for pairing and profile management:
    ```bash
    bluetoothctl
    [bluetooth]# power on
    [bluetooth]# agent on
    [bluetooth]# default-agent
    [bluetooth]# scan on
    # Locate [Corne] device MAC address
    [bluetooth]# pair <MAC>
    [bluetooth]# connect <MAC>
    [bluetooth]# trust <MAC>
    ```
- **Clipboard Combos:** Clipboard combos (`LC(C)`, `LC(V)`, `LC(X)`) in [corne.keymap](file:///home/rvkt/zmk-config/config/corne.keymap) send standard `Control` modifier scancodes, fully compatible with standard Linux terminal emulators, desktop environments, and browsers.

---

## 3. Keymap Engineering

### 3.1 Physical Key Position Matrix
The Corne 42-key layout organizes switches into a 3×6 grid per hand plus 3 thumb keys per hand (total 42 positions, numbered 0 through 41):

```
====================================== CORNE 42-KEY MATRIX ======================================
LEFT HALF (Col 0 to Col 5)                                   RIGHT HALF (Col 6 to Col 11)
Row 0: [ 0]   [ 1]   [ 2]   [ 3]   [ 4]   [ 5]     |     [ 6]   [ 7]   [ 8]   [ 9]   [10]   [11]
Row 1: [12]   [13]   [14]   [15]   [16]   [17]     |     [18]   [19]   [20]   [21]   [22]   [23]
Row 2: [24]   [25]   [26]   [27]   [28]   [29]     |     [30]   [31]   [32]   [33]   [34]   [35]
Thumbs:              [36]   [37]   [38]            |            [39]   [40]   [41]
```

#### Detailed Physical Switch to Index Mapping:
- **Left Half:**
  - Row 0 (Top): `0` = TAB, `1` = Q, `2` = W, `3` = E, `4` = R, `5` = T
  - Row 1 (Home): `12` = ESC / LCTRL, `13` = A (LGUI), `14` = S (LALT), `15` = D (LCTRL), `16` = F (LSHFT), `17` = G
  - Row 2 (Bottom): `24` = LSHFT, `25` = Z, `26` = X, `27` = C, `28` = V, `29` = B
  - Thumbs: `36` = Inner/Left thumb, `37` = Middle thumb, `38` = Outer/Right thumb
- **Right Half:**
  - Row 0 (Top): `6` = Y, `7` = U, `8` = I, `9` = O, `10` = P, `11` = BSPC
  - Row 1 (Home): `18` = H, `19` = J (RSHFT), `20` = K (RCTRL), `21` = L (RALT), `22` = ' (RGUI), `23` = HYPER
  - Row 2 (Bottom): `30` = N, `31` = M, `32` = `,`, `33` = `.`, `34` = `/`, `35` = ESC
  - Thumbs: `39` = Inner/Left thumb, `40` = Middle thumb, `41` = Outer/Right thumb

### 3.2 Mod-Tap & Layer-Tap Behavioral Configuration
The keymap overrides default Zephyr/ZMK node parameters for `&mt` (Mod-Tap) and `&lt` (Layer-Tap) at the top of [corne.keymap](file:///home/rvkt/zmk-config/config/corne.keymap#L41-L52):

```dts
&mt {
    flavor = "tap-preferred";
    tapping-term-ms = <200>;
    quick-tap-ms = <175>;
    require-prior-idle-ms = <150>;
};

&lt {
    flavor = "balanced";
    tapping-term-ms = <200>;
    quick-tap-ms = <175>;
};
```

- **`require-prior-idle-ms = <150>` on `&mt`:** Crucial for fast typing. Prevents accidental modifier triggers when rolling home-row keys unless the user pauses typing for at least 150ms before holding the key.
- **`flavor = "tap-preferred"` on `&mt`:** Favors outputting the character key over the modifier if another key is pressed and released within `tapping-term-ms`.
- **`flavor = "balanced"` on `&lt`:** Balances thumb layer activation with fast tap access.

### 3.3 Home-Row Mod-Tap (GACS System)
Modifiers follow the standard Miryoku **GACS** order (GUI, Alt, Ctrl, Shift) moving inward toward the index finger:

| Hand | Finger | Position | Tap Key | Hold Modifier | ZMK Binding |
|---|---|---|---|---|---|
| **Left** | Pinky | 13 | `A` | Left GUI (Super) | `&mt LGUI A` |
| **Left** | Ring | 14 | `S` | Left Alt | `&mt LALT S` |
| **Left** | Middle | 15 | `D` | Left Control | `&mt LCTRL D` |
| **Left** | Index | 16 | `F` | Left Shift | `&mt LSHFT F` |
| **Right** | Index | 19 | `J` | Right Shift | `&mt RSHFT J` |
| **Right** | Middle | 20 | `K` | Right Control | `&mt RCTRL K` |
| **Right** | Ring | 21 | `L` | Right Alt | `&mt RALT L` |
| **Right** | Pinky | 22 | `'` (SQT) | Right GUI (Super) | `&mt RGUI SQT` |
| **Left (Outer)**| Outer Pinky | 12 | `ESC` | Left Control | `&mt LCTRL ESC` |

### 3.4 Thumb-Key Assignments Across Layers
The 6 thumb keys (positions 36–41) provide layer access and core editing keys:

| Position | Base Layer Binding | Tap Action | Hold Action |
|---|---|---|---|
| **36** (Left Inner) | `&lt MEDIA ESC` | Escape | Layer 3 (`MEDIA`) |
| **37** (Left Middle) | `&lt NAV SPACE` | Space | Layer 1 (`NAV`) |
| **38** (Left Outer) | `&lt MOUSE TAB` | Tab | Layer 6 (`MOUSE`) |
| **39** (Right Inner) | `&lt SYM RET` | Return / Enter | Layer 4 (`SYM`) |
| **40** (Right Middle) | `&lt NUM BSPC` | Backspace | Layer 2 (`NUM`) |
| **41** (Right Outer) | `&lt FUN DEL` | Delete | Layer 5 (`FUN`) |

### 3.5 Layer Definitions (0 through 6)

#### Layer 0: `BASE` (Default Typing Layer)
- QWERTY alpha matrix.
- Home-row GACS mod-taps on both halves.
- Key position 23 (far right home row) bound to `&hyper` macro (GUI + Alt + Ctrl + Shift).
- Left outer col: `&kp TAB` (0), `&mt LCTRL ESC` (12), `&kp LSHFT` (24).
- Right outer col: `&kp BSPC` (11), `&hyper` (23), `&kp ESC` (35).

#### Layer 1: `NAV` (Navigation & Clipboard)
- **Activation:** Hold `SPACE` (thumb pos 37).
- **Right Hand Navigation (Vim arrows):**
  - Pos 18: `&kp LEFT`
  - Pos 19: `&kp DOWN`
  - Pos 20: `&kp UP`
  - Pos 21: `&kp RIGHT`
  - Pos 22: `&kp CAPS`
- **Right Hand Paging & Document Navigation:**
  - Pos 30: `&kp INS`, Pos 31: `&kp HOME`, Pos 32: `&kp PG_DN`, Pos 33: `&kp PG_UP`, Pos 34: `&kp END`
- **Right Hand Top Row (Clipboard & History):**
  - Pos 6: `&kp LC(LS(Z))` (Redo)
  - Pos 7: `&kp LC(V)` (Paste)
  - Pos 8: `&kp LC(C)` (Copy)
  - Pos 9: `&kp LC(X)` (Cut)
  - Pos 10: `&kp LC(Z)` (Undo)
- **Thumbs:** Pos 39: `&kp RET`, Pos 40: `&kp BSPC`, Pos 41: `&kp DEL`.

#### Layer 2: `NUM` (Left-Hand Numeric Keypad)
- **Activation:** Hold `BSPC` (thumb pos 40).
- **Left Hand 10-Key Layout:**
  - Row 0: Pos 1: `&kp LBKT`, Pos 2: `&kp N7`, Pos 3: `&kp N8`, Pos 4: `&kp N9`, Pos 5: `&kp RBKT`
  - Row 1: Pos 13: `&kp SEMI`, Pos 14: `&kp N4`, Pos 15: `&kp N5`, Pos 16: `&kp N6`, Pos 17: `&kp EQUAL`
  - Row 2: Pos 25: `&kp GRAVE`, Pos 26: `&kp N1`, Pos 27: `&kp N2`, Pos 28: `&kp N3`, Pos 29: `&kp BSLH`
  - Thumbs: Pos 36: `&kp DOT`, Pos 37: `&kp N0`, Pos 38: `&kp MINUS`
- **Right Hand:** Mirrored home-row modifiers (`RSHFT`, `RCTRL`, `RALT`, `RGUI`).

#### Layer 3: `MEDIA` (Media, Bluetooth, RGB, Studio Unlock)
- **Activation:** Hold `ESC` (thumb pos 36).
- **Right Hand Row 0 (RGB & Studio):**
  - Pos 6: `&rgb_ug RGB_TOG` (Toggle RGB on/off)
  - Pos 7: `&rgb_ug RGB_EFF` (Cycle effect forward)
  - Pos 8: `&rgb_ug RGB_HUI` (Hue increase)
  - Pos 9: `&rgb_ug RGB_SAI` (Saturation increase)
  - Pos 10: `&studio_unlock` (Unlock ZMK Studio session)
- **Right Hand Row 1 (Audio & Power):**
  - Pos 18: `&ext_power EP_TOG` (Toggle VCC power to external devices)
  - Pos 19: `&kp C_PREV` (Previous Track)
  - Pos 20: `&kp C_VOL_DN` (Volume Down)
  - Pos 21: `&kp C_VOL_UP` (Volume Up)
  - Pos 22: `&kp C_NEXT` (Next Track)
- **Right Hand Row 2 (Connectivity & Bluetooth):**
  - Pos 30: `&out OUT_TOG` (Toggle output between BLE and USB)
  - Pos 31: `&bt BT_SEL 0` (Select Bluetooth Profile 0)
  - Pos 32: `&bt BT_SEL 1` (Select Bluetooth Profile 1)
  - Pos 33: `&bt BT_SEL 2` (Select Bluetooth Profile 2)
  - Pos 34: `&bt BT_SEL 3` (Select Bluetooth Profile 3)
  - Pos 35: `&bt BT_CLR` (Clear bond on active Bluetooth Profile)
- **Thumbs:** Pos 39: `&kp C_STOP`, Pos 40: `&kp C_PP` (Play/Pause), Pos 41: `&kp C_MUTE`.

#### Layer 4: `SYM` (Left-Hand Symbols)
- **Activation:** Hold `ENTER` (thumb pos 39).
- **Left Hand Symbols:**
  - Row 0: Pos 1: `&kp LBRC` (`{`), Pos 2: `&kp AMPS` (`&`), Pos 3: `&kp ASTRK` (`*`), Pos 4: `&kp LPAR` (`(`), Pos 5: `&kp RBRC` (`}`)
  - Row 1: Pos 13: `&kp COLON` (`:`), Pos 14: `&kp DLLR` (`$`), Pos 15: `&kp PRCNT` (`%`), Pos 16: `&kp CARET` (`^`), Pos 17: `&kp PLUS` (`+`)
  - Row 2: Pos 25: `&kp TILDE` (`~`), Pos 26: `&kp EXCL` (`!`), Pos 27: `&kp AT` (`@`), Pos 28: `&kp HASH` (`#`), Pos 29: `&kp PIPE` (`|`)
  - Thumbs: Pos 36: `&kp LPAR` (`(`), Pos 37: `&kp RPAR` (`)`), Pos 38: `&kp UNDER` (`_`)
- **Right Hand:** Mirrored home-row modifiers.

#### Layer 5: `FUN` (Left-Hand Function Keys)
- **Activation:** Hold `DEL` (thumb pos 41).
- **Left Hand Function Matrix:**
  - Row 0: Pos 1: `&kp F12`, Pos 2: `&kp F7`, Pos 3: `&kp F8`, Pos 4: `&kp F9`, Pos 5: `&kp PSCRN`
  - Row 1: Pos 13: `&kp F11`, Pos 14: `&kp F4`, Pos 15: `&kp F5`, Pos 16: `&kp F6`, Pos 17: `&kp SLCK`
  - Row 2: Pos 25: `&kp F10`, Pos 26: `&kp F1`, Pos 27: `&kp F2`, Pos 28: `&kp F3`, Pos 29: `&kp PAUSE_BREAK`
  - Thumbs: Pos 36: `&kp K_APP` (Context Menu), Pos 37: `&kp SPACE`, Pos 38: `&kp TAB`
- **Right Hand:** Mirrored home-row modifiers.

#### Layer 6: `MOUSE` (ZMK Pointing Device Layer)
- **Activation:** Hold `TAB` (thumb pos 38).
- **Right Hand Pointer & Scroll:**
  - Row 1 (Cursor movement): Pos 18: `&mmv MOVE_LEFT`, Pos 19: `&mmv MOVE_DOWN`, Pos 20: `&mmv MOVE_UP`, Pos 21: `&mmv MOVE_RIGHT`
  - Row 2 (Scrolling): Pos 30: `&msc SCRL_LEFT`, Pos 31: `&msc SCRL_DOWN`, Pos 32: `&msc SCRL_UP`, Pos 33: `&msc SCRL_RIGHT`
- **Right Thumbs (Mouse buttons):**
  - Pos 39: `&mkp MCLK` (Middle Click)
  - Pos 40: `&mkp LCLK` (Left Click)
  - Pos 41: `&mkp RCLK` (Right Click)

### 3.6 Combos Configuration
The repository implements 17 combos defined under the Devicetree `/ { combos { ... }; };` node:

| Combo Node Name | Keys Pressed | Key Indices | Output Binding | Timeout | Prior Idle | Category |
|---|---|---|---|---|---|---|
| `combo_bootloader_left` | `TAB` + `Q` | `0 1` | `&bootloader` | 50ms | 150ms | Hardware Bootloader |
| `combo_bootloader_right` | `P` + `BSPC` | `10 11` | `&bootloader` | 50ms | 150ms | Hardware Bootloader |
| `combo_studio_unlock` | `TAB` + `ESC` | `0 12` | `&studio_unlock` | 50ms | 150ms | ZMK Studio Security |
| `combo_caps_word` | `F` + `J` | `16 19` | `&caps_word` | 30ms | 150ms | Tier 1: Essential |
| `combo_esc` | `W` + `E` | `2 3` | `&kp ESC` | 30ms | 150ms | Tier 1: Essential |
| `combo_bspc` | `I` + `O` | `8 9` | `&kp BSPC` | 30ms | 150ms | Tier 1: Essential |
| `combo_del` | `O` + `P` | `9 10` | `&kp DEL` | 30ms | 150ms | Tier 1: Essential |
| `combo_tab` | `S` + `D` | `14 15` | `&kp TAB` | 30ms | 150ms | Tier 2: High Value |
| `combo_enter` | `K` + `L` | `20 21` | `&kp RET` | 30ms | 150ms | Tier 2: High Value |
| `combo_copy` | `X` + `C` | `26 27` | `&kp LC(C)` | 30ms | 150ms | Tier 2: Clipboard |
| `combo_paste` | `C` + `V` | `27 28` | `&kp LC(V)` | 30ms | 150ms | Tier 2: Clipboard |
| `combo_cut` | `X` + `V` | `26 28` | `&kp LC(X)` | 30ms | 150ms | Tier 2: Clipboard |
| `combo_minus` | `J` + `M` | `19 31` | `&kp MINUS` | 40ms | 100ms | Tier 3: Vertical Symbol |
| `combo_underscore` | `H` + `N` | `18 30` | `&kp UNDER` | 40ms | 100ms | Tier 3: Vertical Symbol |
| `combo_equal` | `F` + `V` | `16 28` | `&kp EQUAL` | 40ms | 100ms | Tier 3: Vertical Symbol |
| `combo_grave` | `S` + `X` | `14 26` | `&kp GRAVE` | 40ms | 100ms | Tier 3: Vertical Symbol |
| `combo_semi` | `L` + `'` | `21 22` | `&kp SEMI` | 30ms | 150ms | Tier 3: Symbol |

### 3.7 Macros & Custom Behaviors
The repository defines one custom macro under `/ { macros { ... }; };`:
- **`hyper` Macro:**
  - Node: `hyper: hyper { ... }`
  - Compatible: `"zmk,behavior-macro"`
  - Description: Presses all four modifiers simultaneously (`LGUI`, `LALT`, `LCTRL`, `LSHFT`), waits for key release via `<&macro_pause_for_release>`, and releases all four modifiers.
  - Bound to: Position 23 (far right key of Row 1 on BASE layer).

---

## 4. ZMK Configuration Guidelines

### 4.1 Devicetree (`config/corne.keymap`) Syntax Rules
1. **Required Header Includes:**
   Every keymap file modifying or adding bindings must include the necessary dt-bindings:
   ```dts
   #include <behaviors.dtsi>
   #include <dt-bindings/zmk/bt.h>
   #include <dt-bindings/zmk/ext_power.h>
   #include <dt-bindings/zmk/keys.h>
   #include <dt-bindings/zmk/outputs.h>
   #include <dt-bindings/zmk/pointing.h>
   #include <dt-bindings/zmk/rgb.h>
   ```
2. **Node Scoping:**
   Custom behaviors, combos, macros, and the keymap matrix MUST live inside the root Devicetree scope:
   ```dts
   / {
       combos { compatible = "zmk,combos"; ... };
       macros { ... };
       keymap { compatible = "zmk,keymap"; ... };
   };
   ```
3. **Macro Cell Convention:** All `behavior-macro` nodes MUST declare `#binding-cells = <0>;`.

### 4.2 Kconfig (`config/corne.conf`) Parameters Reference
Below is the exhaustive, verified set of Kconfig options implemented in [config/corne.conf](file:///home/rvkt/zmk-config/config/corne.conf):

```ini
# --- Keyboard Identity ---
CONFIG_ZMK_KEYBOARD_NAME="Corne"

# --- OLED Display (SSD1306 128x32 - Vertical Layout via zmk-nice-oled) ---
CONFIG_ZMK_DISPLAY=y
CONFIG_ZMK_DISPLAY_STATUS_SCREEN_CUSTOM=y
CONFIG_ZMK_DISPLAY_WORK_QUEUE_DEDICATED=y

# --- Power Management & Deep Sleep ---
CONFIG_ZMK_BATTERY_REPORT_INTERVAL=60
CONFIG_ZMK_IDLE_TIMEOUT=600000          # 10 minutes (screen/RGB off)
CONFIG_ZMK_SLEEP=y
CONFIG_ZMK_IDLE_SLEEP_TIMEOUT=900000    # 15 minutes (~20µA deep sleep)

# --- Bluetooth ---
CONFIG_BT_CTLR_TX_PWR_PLUS_8=y          # +8 dBm max transmission power
CONFIG_BT_GATT_ENFORCE_SUBSCRIPTION=n  # Windows battery subscription fix

# --- Key Scan Debounce ---
CONFIG_ZMK_KSCAN_DEBOUNCE_PRESS_MS=5
CONFIG_ZMK_KSCAN_DEBOUNCE_RELEASE_MS=5

# --- Mouse / Pointing Device ---
CONFIG_ZMK_POINTING=y

# --- ZMK Studio (live keymap editing via zmk.studio) ---
CONFIG_ZMK_STUDIO=y
CONFIG_ZMK_STUDIO_LOCKING=n
CONFIG_ZMK_STUDIO_LOCK_ON_DISCONNECT=n

# --- RGB Underglow (27 WS2812 LEDs) ---
CONFIG_ZMK_RGB_UNDERGLOW=y
CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y
CONFIG_ZMK_RGB_UNDERGLOW_EFF_START=3    # Wave/ripple start effect
CONFIG_ZMK_RGB_UNDERGLOW_BRT_MAX=60     # Brightness cap at 60%
CONFIG_ZMK_RGB_UNDERGLOW_EXT_POWER=n    # Powered from regulator, not ext_power
```

### 4.3 Bluetooth & Split Communication Architecture
- **Central (Left):** Manages the BLE connection to the host computer (Windows/Linux/macOS) and coordinates peripheral keystrokes.
- **Peripheral (Right):** Broadcasts solely to the central half. Does not expose a generic Bluetooth pairing profile to the host.
- **Transmission Power:** `CONFIG_BT_CTLR_TX_PWR_PLUS_8=y` sets Nordic nRF52840 radio output to `+8 dBm`. This overcomes signal attenuation from aluminum cases, desk obstruction, or 2.4 GHz WiFi interference.
- **Profiles:** Profiles 0–3 (`BT_SEL 0` to `BT_SEL 3`) allow fast switching across 4 host devices.

### 4.4 Battery & Power Optimization
- **Idle State (10 min):** After 600,000 ms of inactivity, OLED display blanking and RGB auto-off engage (`CONFIG_ZMK_RGB_UNDERGLOW_AUTO_OFF_IDLE=y`).
- **Deep Sleep (15 min):** After 900,000 ms of inactivity, the nice!nano enters ultra-low power System OFF / deep sleep mode, dropping current consumption to ~20 µA.
- **Wakeup:** Pressing any physical switch immediately triggers an external pin interrupt, waking the controller and reconnecting BLE within ~1 second.

### 4.5 ZMK Studio Integration
- **USB-Only RPC Snippet:** ZMK Studio communication is configured via the `studio-rpc-usb-uart` snippet in [build.yaml](file:///home/rvkt/zmk-config/build.yaml) for `corne_left`.
- **Live Remapping:** Navigating to [https://zmk.studio](https://zmk.studio) over a supported browser (Chromium-based with WebSerial enabled) allows real-time key remapping without compiling or flashing firmware.
- **Security & Unlocking:**
  - In `corne.conf`: `CONFIG_ZMK_STUDIO_LOCKING=n` disables mandatory locking on disconnect.
  - In `corne.keymap`: Dedicated unlock combo `TAB` + `ESC` (keys 0 + 12) and key `&studio_unlock` (Layer 3 pos 10) are implemented in case locking is re-enabled.

---

## 5. Build and Debugging

### 5.1 Common Compilation Errors & Historical Pitfalls

#### 1. Board Name Qualifier Failure
- **Error Symptom:**
  ```text
  The selected board is not set up for ZMK and there is a ZMK variant available.
  See https://zmk.dev/blog/2025/12/09/zephyr-4-1
  ```
- **Root Cause:** Using `nice_nano_v2`, `nice_nano`, or `nice_nano_v2//zmk`.
- **Solution:** Always specify `board: nice_nano//zmk` in `build.yaml`.

#### 2. Missing `west.yml` Manifest
- **Error Symptom:**
  ```text
  west: unknown command boards; do you need to run this inside a workspace?
  ```
- **Root Cause:** Missing `config/west.yml`. Upstream GitHub Actions runner cannot initialize the ZMK workspace.
- **Solution:** Ensure [config/west.yml](file:///home/rvkt/zmk-config/config/west.yml) exists with `path: config` and points to `zmkfirmware/zmk:main`.

#### 3. Unsupported / Deprecated Kconfig Options
- **Historical Failure Log:**
  - `CONFIG_LV_DISPLAY_ORIENTATION_PORTRAIT=y` — *Build failed in commit `3c79582`, reverted in `43a9293`.* LVGL display rotation flags must not be applied without explicit driver support in the shield.
  - `CONFIG_ZMK_BLE_EXPERIMENTAL_CONN=y` — *Unrecognized in newer ZMK main.*
  - `CONFIG_ZMK_SPLIT_BLE_CENTRAL_BATTERY_LEVEL_PROXY=y` — *Invalid or unsupported in this branch.*
  - `CONFIG_ZMK_DISPLAY_BLANK_ON_IDLE=y` — *Unrecognized symbol; blanking is managed by idle timeouts.*
- **Rule:** Never introduce speculative Kconfig symbols without confirming they exist in `zmkfirmware/zmk:main/app/Kconfig`.

#### 4. Missing Pointing Subsystem Include
- **Error Symptom:** `undefined symbol 'mmv'` or `undefined symbol 'mkp'`.
- **Solution:** Add `#include <dt-bindings/zmk/pointing.h>` to `corne.keymap` and ensure `CONFIG_ZMK_POINTING=y` is set in `corne.conf`.

#### 5. Matrix Column/Position Bounds Violation
- **Error Symptom:** Devicetree compiler error `key-positions out of range`.
- **Verification:** Corne 42-key positions are strictly `0` through `41`. Any combo with an index `≥ 42` will trigger a build abort.

### 5.2 Bluetooth Pairing & Connection Troubleshooting

#### A. Split Halves Not Communicating (Right Half Silent)
1. Ensure both halves are charged.
2. Connect Left half to computer via USB; observe OLED display. If the peripheral indicator is blinking or disconnected, the right half has lost its BLE bond.
3. Flash `settings_reset-nice_nano-zmk.uf2` to **BOTH halves**:
   - Put Left half in bootloader mode -> flash reset UF2.
   - Put Right half in bootloader mode -> flash reset UF2.
4. Reflash `corne_left-nice_nano-zmk.uf2` to Left half.
5. Reflash `corne_right-nice_nano-zmk.uf2` to Right half.
6. Power on both halves simultaneously within 10 seconds to allow initial split BLE pairing.

#### B. Host Cannot Connect or Pairing Fails
1. On host OS (Linux / Omarchy): Remove the keyboard from `bluetoothctl` (`remove <MAC>`).
2. On keyboard: Switch to the active profile on Layer 3 (`MEDIA`), then press `&bt BT_CLR` (Layer 3 pos 35).
3. Put the host into discovery mode (`bluetoothctl scan on`).
4. Select the matching profile (`BT_SEL 0`) and initiate pairing from host.

---

## 6. Engineering Standards for Gemini & Antigravity

When executing modifications in this repository:

1. **Inspect Before Modifying:**
   - Always read the exact current keymap and config lines before proposing changes.
   - Confirm layer indices and thumb key positions before altering layout definitions.
2. **Minimal & Targeted Diffs:**
   - Never regenerate the entire keymap or configuration when modifying specific keys or combos.
   - Maintain original formatting, comments, indentation, and alignment.
3. **Preserve Working Architecture:**
   - Do not alter `build.yaml` board qualifiers (`nice_nano//zmk`) or snippets unless explicitly requested.
   - Retain essential bootloader and unlock combos (`combo_bootloader_left`, `combo_bootloader_right`, `combo_studio_unlock`).
4. **Never Invent Unsupported Properties:**
   - Verify every Kconfig property against the official ZMK Kconfig documentation.
   - Verify all keycodes against `dt-bindings/zmk/keys.h` and official ZMK documentation.
5. **No Blind Claims:**
   - Never claim a build succeeded without executing it or verifying CI build output.

---

## 7. Antigravity / Gemini Workflow

Follow this standardized 7-step execution process for all firmware feature additions, debugging tasks, and layout refactorings:

```
[1. Inspect]       Read config/corne.keymap, config/corne.conf, and build.yaml
     ↓
[2. Understand]    Trace layer dependencies, combo indices, and hardware flags
     ↓
[3. Root Cause]    Pinpoint exact defect or requirement specification
     ↓
[4. Propose]       Draft the smallest targeted Devicetree or Kconfig diff
     ↓
[5. Implement]     Execute precise file edit using edit_file tool
     ↓
[6. Validate]      Verify Devicetree syntax, brackets, semicolons, and matrix indices
     ↓
[7. Report]        Detail changed files, modified lines, and flashing instructions
```

---

## 8. Project Constraints & Verification Matrix

### 8.1 Confirmed vs Unverified Inventory

| Dimension | Attribute | Status | Source / Evidence |
|---|---|---|---|
| **Board Identifier** | `nice_nano//zmk` | CONFIRMED | `build.yaml`, Commit `e8f4a06` |
| **Shield Target** | `corne_left`, `corne_right` | CONFIRMED | `build.yaml` |
| **Key Matrix Count** | 42 keys (21 per half) | CONFIRMED | `config/corne.keymap`, `corne.conf` |
| **Layers Count** | 7 layers (0 to 6) | CONFIRMED | `config/corne.keymap` |
| **Display Hardware** | SSD1306 128x32 OLED | CONFIRMED | `config/corne.conf`, `build.yaml` |
| **Display Rotation** | Default horizontal | CONFIRMED | Portrait mode removed in commit `43a9293` |
| **RGB Quantity** | 27 WS2812 LEDs | CONFIRMED | `config/corne.conf`, `README.md` |
| **ZMK Studio** | USB-UART RPC enabled | CONFIRMED | `build.yaml`, `config/corne.conf` |
| **Host System** | Linux / Omarchy | CONFIRMED | Repository prompt specification |
| **Battery Capacity** | Exact mAh (e.g. 110mAh) | NOT VERIFIED | Hardware specific, not stored in git |
| **Switch Type** | Choc vs Cherry MX | NOT VERIFIED | Hardware specific, transparent to firmware |

### 8.2 Critical Operational Constraints
- **Flash Space & Memory:** nice!nano v2 nRF52840 has 1 MB flash. Enabling both ZMK Studio, Display, and RGB increases binary footprint; avoid adding unused heavy drivers or fonts.
- **ZMK Studio over BLE:** ZMK Studio live remapping currently operates over USB UART RPC only in this build (`snippet: studio-rpc-usb-uart`). It is NOT supported over Bluetooth without custom BLE studio configurations.
- **OLED Orientation:** Setting `CONFIG_LV_DISPLAY_ORIENTATION_PORTRAIT=y` caused build failures and was removed. Do not re-enable without custom display driver definitions.
