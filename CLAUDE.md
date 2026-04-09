# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

ZMK firmware configuration for **LalaPadmini** — a split wireless keyboard using two Seeeduino XIAO BLE controllers. Each half has a Cirque Pinnacle trackpad (SPI) and an EC11 rotary encoder. The right side is the BLE central.

## Build

Firmware is built via GitHub Actions. Push to any branch triggers `.github/workflows/build.yml`, which delegates to `zmkfirmware/zmk/.github/workflows/build-user-config.yml@main`. There is no local build — CI produces the firmware artifacts.

`build.yaml` defines three build targets:
- `seeeduino_xiao_ble` + `lalapadmini_right rgbled_adapter` (central, with ZMK Studio RPC)
- `seeeduino_xiao_ble` + `lalapadmini_left rgbled_adapter`
- `seeeduino_xiao_ble` + `settings_reset`

## Key Files

- **`config/lalapadmini.keymap`** — keymap definition (8 layers, combos, macros, sensor bindings). This is the most frequently edited file.
- **`config/lalapadmini.conf`** — Kconfig options (Bluetooth, sleep, battery, trackpad, encoder, ZMK Studio, RGB LED widget).
- **`config/west.yml`** — West manifest pulling ZMK main, `ShiniNet/cirque-input-module`, and `caksoylar/zmk-rgbled-widget`.
- **`config/boards/shields/lalapadmini/`** — shield hardware definitions (matrix, SPI trackpad, encoders, split input listeners, physical layout).

## Architecture

### Split Keyboard Design
- Right half = BLE central (`ZMK_SPLIT_ROLE_CENTRAL`), left half = peripheral.
- Matrix: 10 columns × 4 rows (5 cols per side, col-offset 5 on right). Bottom row has gaps (no col 2 or 7) → 38 keys total.
- Diode direction: col2row.

### Trackpad (Cirque Pinnacle)
- Connected via SPI on each half (`&xiao_spi`, `cs-gpios` on P1.10).
- Left side: `glidepoint_split_L` feeds data to central; right side: `glidepoint_listener_R` reads directly.
- Input processors in the keymap apply coordinate transforms (swap XY, invert), scaling, and temporary layer activation (`MOUSE_R_LAYER`/`MOUSE_L_LAYER`).
- Scroll mode activated on `SCROLL_R_LAYER`/`SCROLL_L_LAYER` via `zip_xy_to_scroll_mapper`.

### Layer Structure (keymap)
| # | Name | Purpose |
|---|------|---------|
| 0 | DEFAULT | QWERTY base with mod-taps (Ctrl, Shift, Cmd, Alt) |
| 1 | SYMBOL | Symbols and programming punctuation |
| 2 | NOT_USE | Tertiary (navigation/symbols, currently named "not use") |
| 3 | NUMBER | Numbers, arrows, screenshot shortcuts |
| 4 | MOUSE_R | Right trackpad mouse buttons/scroll |
| 5 | SCROLL_R / SYSTEM | Scroll layer (right) / conditional system layer (layers 1+2 → 5) |
| 6 | MOUSE_L | Left trackpad mouse buttons |
| 7 | SCROLL_L | Scroll layer (left) |

### Conditional Layer
Layers 1 + 2 active simultaneously → activates layer 5 (system layer with bootloader access).

## External Modules
- `zmkfirmware/zmk` (main) — core ZMK firmware
- `ShiniNet/cirque-input-module` — Cirque trackpad driver/input module
- `caksoylar/zmk-rgbled-widget` — RGB LED battery/layer indicator widget
