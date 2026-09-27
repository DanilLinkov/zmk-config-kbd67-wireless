# KBD67 Wireless — ZMK firmware

This firmware is for a **KBD67 Lite R2** (KBD67 MKII RGB **V2** PCB) that has been converted to Bluetooth. The original ATmega32U4 chip is removed. The PCB's 5 × 15 key matrix is then wired by hand to a nice!nano-compatible **nRF52840 SuperMini**, which runs [ZMK](https://zmk.dev) v0.3.

- It pairs over Bluetooth as **"KBD67 Wireless"** (up to 5 devices) and also works over USB.
- The per-key RGB is not used. Its driver chip stays unpowered, so it doesn't drain the battery.
- [ZMK Studio](https://zmk.studio) is enabled, so you can remap keys live over USB, much like VIA.

## Getting the firmware

Every push triggers a GitHub Actions build. Open **Actions**, then the latest run, and download **firmware** under **Artifacts**. The zip contains:

| File | What it's for |
|---|---|
| `kbd67w-nice_nano_v2-zmk.uf2` | The keyboard |
| `settings_reset-nice_nano_v2-zmk.uf2` | Wipes Bluetooth pairings and settings. Flash it, then flash the keyboard firmware again |

## Flashing

1. Plug the controller into USB.
2. Quickly short **RST** to **GND** twice. Tweezers work for this, since the SuperMini has no reset button. A drive called `NICENANO` appears.
3. Drag the `.uf2` file onto that drive. The controller reboots by itself when the copy finishes.

Once the keyboard is working, **Fn + \\** does the same job as step 2.

## Testing the controller before soldering

1. Flash the keyboard firmware.
2. Pair **KBD67 Wireless** in your computer's Bluetooth settings.
3. Touch two pads together with tweezers:
   - `031` + `006` should type **Esc**.
   - `031` + `008` should type **1**.
   - `029` + `008` should type **Q**.

Each row pad combined with each column pad simulates one key.

## Wiring

The controller pads are the numbers printed on the SuperMini. ZMK drives the columns, and each row reads through the key's diode (COL2ROW).

| Controller pad | Matrix line | Keys on this line (number-row key for columns) |
|---|---|---|
| `031` | R0 | Number row: Esc … Home |
| `029` | R1 | Tab row: Tab … PgUp |
| `002` | R2 | Caps row: Caps … PgDn |
| `115` | R3 | Shift row: Shift … End |
| `113` | R4 | Bottom row: Ctrl … Right |
| `006` | C0 | Esc |
| `008` | C1 | 1 |
| `017` | C2 | 2 |
| `107` | C3 | 3 (moved from `020`, which is dead on this SuperMini) |
| `022` | C4 | 4 |
| `024` | C5 | 5 |
| `100` | C6 | 6 |
| `011` | C7 | 7 |
| `104` | C8 | 8 |
| `106` | C9 | 9 |
| `111` | C10 | 0 |
| `010` | C11 | - |
| `009` | C12 | = |
| `101` | C13 | Backspace |
| `102` | C14 | Home |

- **Spare pad:** none. `107` now carries C3.
- **Where to solder on the PCB** (measured with a multimeter):
  - **Columns:** the top end of one of that column's diodes (the end further from you, with the PCB face down and the number row nearest you).
  - **Rows:** the right tab of one hot-swap socket in that row.
  - Confirm every joint with a multimeter continuity test before closing the case.
- **Power:** battery − → `B-`; battery + → slide switch → `B+`. The switch has to be on for the battery to charge.
- **Charging:** bridge the `BOOST` jumper for 300 mA charging. Only do this with batteries over 500 mAh. This build uses 1250 mAh.
- **Matrix map:** key positions follow QMK `keyboards/kbdfans/kbd67/mkiirgb/v2`. Enter is on `RC(2,13)` and right Alt is on `RC(4,8)`.

## Keymap

**Base** is the stock KBD67 layout. **Fn** adds the following:

| Keys | Action |
|---|---|
| Fn + Esc | `` ` `` |
| Fn + 1 … = | F1 … F12 |
| Fn + Backspace | Delete |
| Fn + Q / W / E / R / T | Switch to Bluetooth device 1 / 2 / 3 / 4 / 5 |
| Fn + Y | Forget the current Bluetooth device |
| Fn + U / I | Send keys over USB / Bluetooth |
| Fn + P / [ / ] | Print Screen / Scroll Lock / Pause |
| Fn + \\ | Bootloader (for flashing) |
| Fn + Enter | Unlock ZMK Studio |
| Fn + Space | Play/Pause |
| Fn + Up / Down | Volume up / down |
| Fn + End | Mute |
| Fn + Left / Right | Previous / next track |

- **Remapping:** open [zmk.studio](https://zmk.studio) in Chrome or Edge, connect over USB, and press **Fn + Enter** to unlock.
- **Sleep:** the keyboard sleeps after 30 minutes idle. The first keypress only wakes it and isn't typed.

## Settings worth knowing

- **`nfct-pins-as-gpios` on `&uicr`**, in the shield overlay: frees pads `009` and `010` to use as key columns.
- **`CONFIG_CLOCK_CONTROL_NRF_K32SRC_RC=y`**, in `config/kbd67w.conf`: works around unreliable crystals on some SuperMini clones. You can remove it on a genuine nice!nano.
