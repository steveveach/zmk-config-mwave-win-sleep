# Kinesis mWave — Windows Sleep Macro (ZMK Firmware). Created by Microsoft Copilot and implemented by Steve Veach

A custom ZMK firmware for the [Kinesis mWave keyboard](https://kinesis-ergo.com/keyboards/mwave-keyboard/) that adds a **Windows Sleep** function via the Fn layer.

## The Problem

The Kinesis mWave has no dedicated sleep key, and there is no way to add one through the official [Clique configurator](https://clique.kinesis-ergo.com/). ZMK's native `K_SLEEP` / `SYSTEM_SLEEP` HID usage code is non-functional due to an upstream bug ([zmk-firmware #1077](https://github.com/zmkfirmware/zmk/issues/1077)), so the standard approach doesn't work either.

## The Solution

This fork adds a **ZMK macro** that sends the Windows keyboard shortcut sequence for sleep:

1. **Win+X** — opens the Power User menu
2. **U** — selects "Shut down or sign out"
3. **S** — selects "Sleep"

The macro is bound to the **S key on the Fn layer**, so pressing **Fn → S** puts your Windows PC to sleep. All other keys on both the base and Fn layers remain identical to the stock Kinesis firmware.

> **Note:** This macro targets the Windows Power User menu and only works on Windows 10 and Windows 11.

## What Changed vs. Stock Firmware

Only two changes were made to `config/mwave_33.keymap` relative to the [upstream Kinesis repo](https://github.com/kinesis-corp/zmk-config-mwave):

1. **Added** a `macros` block defining `win_sleep` (sends Win+X → U → S with appropriate timing delays)
2. **Changed one key** in the Fn (`raise`) layer: the S key position changed from `&kp S` to `&win_sleep`

No other files were modified. The build configuration, workflow, and all other keymap layers are unchanged from stock.

## Usage

1. Press the **Fn toggle key** (top-right of the function row)
2. Press **S**
3. Your PC will enter sleep mode
4. After waking, press the **Fn toggle key** again to return to the base layer

## Build Instructions

This fork uses GitHub Actions to compile the firmware automatically.

### Using a Pre-Built Artifact

1. Go to the [Actions tab](../../actions) in this repository
2. Click the most recent successful workflow run (green checkmark)
3. Scroll down to **Artifacts**
4. Download the `ansi-windows` artifact (`.zip` file)
5. Unzip it — the `.uf2` firmware file is inside

### Building from Source (Your Own Fork)

1. Fork this repository to your own GitHub account
2. Go to the **Actions** tab in your fork
3. Click **"I understand my workflows, go ahead and enable them"**
4. Click the workflow name in the left sidebar → **"Run workflow"** → **Branch: main** → **"Run workflow"**
5. Wait ~5 minutes for the build to complete
6. Download the artifact as described above

## Flash Instructions

1. **Backup current firmware:** Double-press the reset pinhole on the bottom of the keyboard with a paperclip. A USB drive will appear. Copy `CURRENT.UF2` from that drive to a safe location on your PC.
2. **Flash:** Drag and drop the new `ansi-windows.uf2` file onto the USB drive.
3. **Ignore the error:** Windows will show an "Interrupted Action" error (error `0x80070022`). This is normal — the keyboard's UF2 bootloader reboots immediately after receiving the firmware, causing Windows to think the copy failed. Click **Cancel**. The flash was successful.
4. **Test:** Save all open work, then press Fn → S. Your PC should sleep.

## Rollback

To revert to stock firmware or your previous firmware:

1. Double-press the reset pinhole to enter bootloader mode
2. Drag and drop your backed-up `CURRENT.UF2` file onto the USB drive
3. Dismiss the error — the keyboard will reboot with the previous firmware

## Technical Details

- **Macro timing:** `wait-ms = 100`, `tap-ms = 50` — provides reliable sequencing for the Win+X power menu
- **Fn layer key position:** Position 48 (S key) in the `raise` layer
- **Why not a combo?** A combo (e.g., GUI+S on the base layer) was tested and rejected. ZMK combos queue the first key for up to `timeout-ms`, adding perceptible input lag to every Win+\<key\> shortcut. The Fn layer approach has zero impact on normal typing.
- **Why not native K_SLEEP?** ZMK's HID sleep usage code is not functional. See [zmk-firmware #1077](https://github.com/zmkfirmware/zmk/issues/1077). The macro workaround is the only reliable method currently available.

## Upstream Repository

This is a fork of [kinesis-corp/zmk-config-mwave](https://github.com/kinesis-corp/zmk-config-mwave).

## License

Same license as the upstream Kinesis ZMK configuration repository.
