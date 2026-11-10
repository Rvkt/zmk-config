# ZMK Firmware Configuration for Corne Keyboard (42-Key)

This repository contains the custom ZMK firmware configuration for a **42-key split Corne keyboard** (CRKBD with nice!nano v2 microcontrollers).

---

## 🗺️ Layer Map

The layout is arranged into **4 layers** using a **Tri-Layer** system:
- Hold left thumb `LOWER` for numbers and arrow navigation.
- Hold right thumb `RAISE` for symbols and programming brackets.
- Hold **both** `LOWER` + `RAISE` at the same time to open the **Adjust Layer** (Bluetooth, media, and bootloader).

---

### Layer 0: Base (QWERTY)
Standard typing layer with dedicated modifier keys and thumb cluster.

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

### Layer 1: Lower (Numbers & Navigation)
*Trigger:* Hold **`LWR`** (Left thumb middle). Numbers on top, navigation cluster on the right hand.

```text
Left Half                                      Right Half
┌─────┬─────┬─────┬─────┬─────┬─────┐        ┌─────┬─────┬─────┬─────┬─────┬─────┐
│ TAB │  1  │  2  │  3  │  4  │  5  │        │  6  │  7  │  8  │  9  │  0  │BSPC │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│CTRL │ F1  │ F2  │ F3  │ F4  │ F5  │        │ LFT │ DWN │ UP  │ RGT │     │     │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│SHFT │ F6  │ F7  │ F8  │ F9  │ F10 │        │HOME │PG_DN│PG_UP│ END │     │     │
└─────┴─────┴─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┴─────┴─────┴─────┘
                  │ GUI │ === │ SPC │        │ ENT │ === │ ALT │
                  └─────┴─────┴─────┘        └─────┴─────┴─────┘
```

---

### Layer 2: Raise (Symbols & Code)
*Trigger:* Hold **`RSE`** (Right thumb middle). Pair-matched brackets `()`, `[]`, `{}`, quotes, and operators.

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
*Trigger:* Hold **both `LWR` + `RSE`** at the same time.

```text
Left Half                                      Right Half
┌─────┬─────┬─────┬─────┬─────┬─────┐        ┌─────┬─────┬─────┬─────┬─────┬─────┐
│BOOT │RESET│     │     │     │     │        │     │PREV │PLAY │NEXT │VOL+ │BOOT │
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│BTCLR│ BT0 │ BT1 │ BT2 │ BT3 │ BT4 │        │     │     │     │MUTE │VOL- │RESET│
├─────┼─────┼─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┼─────┼─────┼─────┤
│OUTTG│OUTBL│OUTUS│     │     │     │        │     │     │     │     │     │     │
└─────┴─────┴─────┼─────┼─────┼─────┤        ├─────┼─────┼─────┴─────┴─────┴─────┘
                  │     │ === │     │        │     │ === │     │
                  └─────┴─────┴─────┘        └─────┴─────┴─────┘
```
* **`BOOT`**: Puts that half into bootloader/flashing mode (no physical reset button needed).
* **`RESET`**: Soft resets the microcontroller.
* **`BT0`–`BT4`**: Switch between 5 paired devices.
* **`BTCLR`**: Clears the current Bluetooth profile bonding keys.
* **`OUTTG`**: Toggle between USB and Bluetooth output.

---

## ⚡ Step-by-Step Build & Flash Guide

### Step 1: Automatic Build via GitHub Actions
Every time you push changes to the `main` branch of this repository, GitHub Actions automatically compiles the firmware:
1. Open [https://github.com/Rvkt/zmk-config/actions](https://github.com/Rvkt/zmk-config/actions).
2. Click on the latest workflow run: **Build ZMK Firmware**.
3. Scroll down to the **Artifacts** section at the bottom.
4. Download the `firmware.zip` archive.
5. Extract the downloaded zip file. You will see:
   - `corne_left-nice_nano_v2-zmk.uf2`
   - `corne_right-nice_nano_v2-zmk.uf2`
   - `settings_reset-nice_nano_v2-zmk.uf2` *(Emergency reset firmware)*
   *(Plus the `_nice_view` variants if your board uses nice!view screens)*

### Step 2: Flash the Left Half (Central)
1. Connect the **Left half** to your computer via USB-C.
2. Double-tap the physical reset button on the nice!nano controller (or press the top-left `BOOT` key on the Adjust layer).
3. A mass storage device named **`NICENANO`** will appear on your computer (like a USB drive).
4. Drag and drop `corne_left-nice_nano_v2-zmk.uf2` onto the `NICENANO` drive.
5. The drive will automatically disconnect and the board will restart with the new firmware.

### Step 3: Flash the Right Half (Peripheral)
1. Unplug the Left half.
2. Connect the **Right half** to your computer via USB-C.
3. Double-tap the reset button on the Right half.
4. Drag and drop `corne_right-nice_nano_v2-zmk.uf2` onto the `NICENANO` drive.
5. The drive will disconnect and reboot.

### Step 4: Reconnect Wirelessly
1. Unplug the USB cable from both halves.
2. Switch **ON** the battery power switch on the **Left half** first.
3. Switch **ON** the battery power switch on the **Right half**.
4. Both halves will sync with each other, and the Left half will automatically reconnect to your computer over Bluetooth.

---

## 🎨 Visual Keymap Editor (In Your Browser)

If you prefer to edit keys graphically without touching code:
1. Open [Nick Coutsos Keymap Editor](https://nickcoutsos.github.io/keymap-editor/).
2. Click **Authorize** to connect your GitHub account.
3. Select your `Rvkt/zmk-config` repository.
4. Click and change any key visually.
5. Hit **Save** / **Commit**—it will push directly to GitHub and build your new firmware automatically!

---

## 🛠️ Troubleshooting & Firmware Recovery

If the two halves refuse to communicate with each other, or if Bluetooth pairing gets stuck/corrupted:
1. Double-tap reset on the **Left half** and flash `settings_reset-nice_nano_v2-zmk.uf2`.
2. Double-tap reset on the **Right half** and flash `settings_reset-nice_nano_v2-zmk.uf2`.
3. Re-flash the regular firmware (`corne_left...uf2` on the left, `corne_right...uf2` on the right).
4. Remove/forget "Corne" from your computer's Bluetooth settings and re-pair fresh.

