# Vibe4 — 4-key Vibe Coding Macropad (ZMK / Nice!Nano)

A 4-key BLE macropad built on **Nice!Nano v2 (nRF52840)** with [ZMK firmware](https://zmk.dev/).
Tuned for "vibe coding" on **macOS**: summon AI, send, interrupt, switch.

## Hardware
- MCU: Nice!Nano v2 (Pro Micro pin-compatible)
- 4 mechanical switches wired direct (no matrix, no diodes) between GPIO and GND:
  - K1 ↔ `D18`
  - K2 ↔ `D19`
  - K3 ↔ `D20`
  - K4 ↔ `D21`
- Optional: 3.7V LiPo on `BAT+`/`GND`, slide switch on `BAT+`/`RAW`.

## Default keymap (BASE)
| Key | Binding | Use |
|-----|---------|-----|
| K1  | `Fn` (momentary layer 1) | Hold for FN layer (Bluetooth controls) |
| K2  | `Ctrl+C` | Interrupt terminal / dev server |
| K3  | `Cmd+K`  | Open AI command palette (Cursor / Warp / Raycast / VS Code) |
| K4  | `Cmd+Enter` | Submit (Cursor / Copilot Chat / ChatGPT) |

## FN layer (hold K1)
| Key | Binding | Use |
|-----|---------|-----|
| K2  | `BT_CLR` | Clear current Bluetooth pairing |
| K3  | `BT_NXT` | Cycle to next Bluetooth host |
| K4  | `BT_SEL 0` | Force Bluetooth profile 0 |

## Build
This repo is a standard zmk-config. Push to GitHub and the bundled GitHub Actions
workflow (`.github/workflows/build.yml`) will build `vibe4-nice_nano_v2-zmk.uf2`.

Flash by double-tapping `RST` on Nice!Nano (mounts as `NICENANO` drive) and
dragging the `.uf2` file onto it.

## Customize
Edit `config/vibe4.keymap`. References:
- ZMK behaviors: https://zmk.dev/docs/behaviors
- Keycodes: https://zmk.dev/docs/codes
