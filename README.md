# ZMK Firmware Configuration for Corne Keyboard (42-Key)

This repository contains the production-grade custom ZMK firmware configuration for a **42-key split Corne keyboard** (CRKBD with nice!nano v2 microcontrollers).

---

## 🗺️ Layer Map

The layout is arranged into **4 layers** using an intelligent **Tri-Layer** system:
- Hold left thumb `LWR` for numbers, function keys, and navigation.
- Hold right thumb `RSE` for pair-matched symbols and programming brackets.
- Hold **both** `LWR` + `RSE` at the same time to activate the **Adjust Layer** (Bluetooth profiles, media controls, and reset).

---

### Layer 0: Base (QWERTY)
Standard typing layer with dedicated modifier keys and ergonomic thumb cluster.

```text
Left Half                                      Right Half
┌─────┬─────┬─────┬─────┬─────┬─────┐        ┌─────┬─────┬─────┬─────┬─────┬─────┐
│ TAB │  Q  │  W  │  E  │  R  │  T  │        │  Y  │  U  │  I  │  O  │  P  │BSPC │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│CTRL │  A  │  S  │  D  │  F  │  G  │        │  H  │  J  │  K  │  L  │  ;  │  '  │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│SHFT │  Z  │  X  │  C  │  V  │  B  │        │  N  │  M  │  ,  │  .  │  /  │ ESC │
└─────┴─────┴─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┴─────┴─────┴─────┘
                  │ GUI │ LWR │ SPC │        │ ENT │ RSE │ ALT │
                  └─────┴─────┴─────┘        └─────┴─────┴─────┘
```

---

### Layer 1: Lower (Numbers, Navigation & Function Keys)
*Trigger:* Hold **`LWR`** (Left thumb middle). Numbers on top, complete F1–F12 keys, arrow cluster, navigation keys, Delete, and PrintScreen.

```text
Left Half                                      Right Half
┌─────┬─────┬─────┬─────┬─────┬─────┐        ┌─────┬─────┬─────┬─────┬─────┬─────┐
│ TAB │  1  │  2  │  3  │  4  │  5  │        │  6  │  7  │  8  │  9  │  0  │ DEL │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│CTRL │ F1  │ F2  │ F3  │ F4  │ F5  │        │ LFT │ DWN │ UP  │ RGT │ F11 │ F12 │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│SHFT │ F6  │ F7  │ F8  │ F9  │ F10 │        │HOME │PG_DN│PG_UP│ END │PSCRN│CAPS │
└─────┴─────┴─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┴─────┴─────┴─────┘
                  │ GUI │ === │ SPC │        │ ENT │ === │ ALT │
                  └─────┴─────┴─────┘        └─────┴─────┴─────┘
```

---

### Layer 2: Raise (Symbols & Code)
*Trigger:* Hold **`RSE`** (Right thumb middle). Pair-matched brackets `()`, `[]`, `{}`, quotes, and operators vertically aligned for muscle memory.

```text
Left Half                                      Right Half
┌─────┬─────┬─────┬─────┬─────┬─────┐        ┌─────┬─────┬─────┬─────┬─────┬─────┐
│ TAB │  !  │  @  │  #  │  $  │  %  │        │  ^  │  &  │  *  │  (  │  )  │BSPC │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│CTRL │     │     │     │     │     │        │  -  │  =  │  [  │  ]  │  \  │  `  │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│SHFT │     │     │     │     │     │        │  _  │  +  │  {  │  }  │ "|" │  ~  │
└─────┴─────┴─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┴─────┴─────┴─────┘
                  │ GUI │ === │ SPC │        │ ENT │ === │ ALT │
                  └─────┴─────┴─────┘        └─────┴─────┴─────┘
```

---

### Layer 3: Adjust (Bluetooth, Media & Bootloader)
*Trigger:* Hold **both `LWR` + `RSE`** at the same time. The home row safely houses device profile switches without risk of accidental unpairing.

```text
Left Half                                      Right Half
┌─────┬─────┬─────┬─────┬─────┬─────┐        ┌─────┬─────┬─────┬─────┬─────┬─────┐
│BOOT │RESET│     │     │     │     │        │     │PREV │PLAY │NEXT │VOL+ │BOOT │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│     │ BT0 │ BT1 │ BT2 │ BT3 │ BT4 │        │     │     │     │MUTE │VOL- │RESET│
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│OUTTG│OUTBL│OUTUS│     │BTCLR│BTALL│        │     │     │     │     │     │     │
└─────┴─────┴─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┴─────┴─────┴─────┘
                  │     │ === │     │        │     │ === │     │
                  └─────┴─────┴─────┘        └─────┴─────┴─────┘
```
* **`BOOT`**: Reboots that half directly into UF2 bootloader mode (USB drive appears).
* **`RESET`**: Soft resets the microcontroller.
* **`BT0`–`BT4`**: Switch between 5 paired host devices.
* **`BTCLR`**: Clears bonding keys on the currently selected Bluetooth profile.
* **`BTALL`**: Clears bonding keys across all 5 profiles.
* **`OUTTG` / `OUTBL` / `OUTUS`**: Toggle output between USB cable and Bluetooth.

---

## ⚡ Instant Bootloader Combos (Zero-Friction Flashing)

No need to open keyboard cases, fiddle with paperclips, or stress tiny physical reset switches! This config features **hardware combos** evaluated locally on each half:

| Target Half | Combo Keys | Description |
|:---|:---|:---|
| **Left Half** | Press **`TAB` + `Q`** together | Immediately opens `NICENANO` drive on Left half |
| **Right Half** | Press **`P` + `BSPC`** together | Immediately opens `NICENANO` drive on Right half |

> [!TIP]
> The Right half combo works even when the Left half is powered off or disconnected. Simply plug the Right half into your computer via USB-C and hit **`P` + `BSPC`**.

---

## 📦 Firmware File Selection Matrix

When downloading artifacts from GitHub Actions (`firmware.zip`), choose the file that matches your hardware:

| Hardware Configuration | Left Half File (Central) | Right Half File (Peripheral) |
|:---|:---|:---|
| **Standard Corne** *(OLED SSD1306 or No Display)* | `corne_left-nice_nano_v2-zmk.uf2` | `corne_right-nice_nano_v2-zmk.uf2` |
| **Corne with nice!view** *(E-Paper Displays)* | `corne_left_nice_view-nice_nano_v2-zmk.uf2` | `corne_right_nice_view-nice_nano_v2-zmk.uf2` |
| **Recovery / Flash Wipe** | `settings_reset-nice_nano_v2-zmk.uf2` *(Flash to both halves)* |

---

## 🚀 Step-by-Step Build & Flash Guide

### Step 1: Automatic Build via GitHub Actions
Every time you push changes to `main` (or run manually via Workflow Dispatch):
1. Open [https://github.com/Rvkt/zmk-config/actions](https://github.com/Rvkt/zmk-config/actions).
2. Click on the latest workflow run: **Build ZMK Firmware**.
3. Under the **Artifacts** section at the bottom, download `firmware.zip`.
4. Extract the zip archive on your computer.

### Step 2: Flash the Left Half (Central)
1. Connect the **Left half** to your computer via USB-C.
2. Trigger bootloader mode by pressing **`TAB` + `Q`** together (or double-tapping the physical reset button).
3. A mass storage drive named **`NICENANO`** will appear on your computer.
4. Drag and drop your chosen left `.uf2` file onto the `NICENANO` drive.
5. The drive will automatically disconnect and the left half reboots.

### Step 3: Flash the Right Half (Peripheral)
1. Unplug the Left half.
2. Connect the **Right half** to your computer via USB-C.
3. Trigger bootloader mode by pressing **`P` + `BSPC`** together (or double-tapping the physical reset button).
4. The **`NICENANO`** drive will appear.
5. Drag and drop your chosen right `.uf2` file onto the `NICENANO` drive.
6. The drive will disconnect and reboot.

### Step 4: Reconnect Wirelessly
1. Unplug the USB cable.
2. Turn **ON** the battery switch on the **Right half**.
3. Turn **ON** the battery switch on the **Left half**.
4. Both halves will pair wirelessly within 2–3 seconds, and the Left half will connect to your computer over Bluetooth.

---

## 🛠️ Bulletproof Troubleshooting & Split Recovery

If the two halves refuse to talk to each other or Bluetooth bonding is corrupted:

1. Connect the **Left half** via USB. Double-tap reset (or press `TAB`+`Q`) and flash `settings_reset-nice_nano_v2-zmk.uf2`.
2. **CRITICAL:** Wait **10 seconds** while powered so the controller can erase the flash partition.
3. Connect the **Right half** via USB. Double-tap reset (or press `P`+`BSPC`) and flash `settings_reset-nice_nano_v2-zmk.uf2`. Wait **10 seconds**.
4. Re-flash the regular firmware onto the **Right half** (`corne_right...uf2`).
5. Re-flash the regular firmware onto the **Left half** (`corne_left...uf2`).
6. Unplug USB, turn on both battery switches within 5 seconds of each other. They will re-sync and form a fresh split bond.
7. Open your computer's Bluetooth settings, **Remove / Forget** any previous "Corne" device, and pair fresh!

---

## 🎨 Visual Keymap Editor (Web-based)

If you prefer to edit keys graphically:
1. Open [Nick Coutsos Keymap Editor](https://nickcoutsos.github.io/keymap-editor/).
2. Authorize GitHub and select `Rvkt/zmk-config`.
3. Visually customize any key or combo.
4. Hit **Save / Commit**—GitHub Actions compiles your new firmware automatically!
