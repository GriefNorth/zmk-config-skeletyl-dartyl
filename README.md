# ZMK Config — Skeletyl (nice!nano, wireless flex-PCB wiring)

ZMK firmware for a [Skeletyl](https://bastardkb.com/) rebuilt as a wireless split with nice!nano
controllers (two halves, Bluetooth), using the BastardKB wireless flexible-PCB matrix layout.

## Hardware

- 2x nice!nano (this config builds against `nice_nano//zmk`, revision 2.0.0)
- Left half = central, right half = peripheral
- Matrix is the **wireless flex-PCB layout** (`row2col`), shared with e.g.
  [280Zo/charybdis-wireless-mini-zmk-firmware](https://github.com/280Zo/charybdis-wireless-mini-zmk-firmware)
  and the [s6t/zmk-shield-skeletyl](https://github.com/s6t/zmk-shield-skeletyl) flex-PCB shield.

### Matrix wiring (must match your physical build)

| | pro_micro pin |
|---|---|
| row 0 | 18 |
| row 1 | 5 |
| row 2 | 4 |
| row 3 | 9 |

| col | left | right |
|---|---|---|
| 0 / 5 | 20 | 8 |
| 1 / 6 | 10 | 7 |
| 2 / 7 | 6 | 6 |
| 3 / 8 | 7 | 10 |
| 4 / 9 | 8 | 20 |

- `diode-direction = "row2col"`
- right half overlays the transform with `col-offset = <5>`

Thumb keys live on row 3 and map to `RC(3,2) RC(3,3) RC(3,0)` (left) /
`RC(3,9) RC(3,6) RC(3,7)` (right).

## Troubleshooting — "keys don't register / wrong keys"

**If only SOME keys work, or pressing `S` outputs a different character (or nothing), your
matrix pin assignment does not match how the board is physically wired.** This repo went
through exactly that.

### What happened here

The config was originally copied from the **wired** Skeletyl variant: `col2row`,
rows `15/14/16/10`, cols `5/6/7/8/9`. But the physical board used the **wireless flex-PCB**
layout above. On the wired layout `S` should be `RC(1,1)`; the actual flex-PCB build puts `S`
on row1 = pin 5 and col1 = pin 10. With the wrong pin mapping, pressing `S` produced
`Not found in transform: row: 3, col: 0, pressed: true` in the log and nothing was ever sent.

### How to figure out your own wiring mismatch

1. Build one half with USB logging: in `build.yaml`, add `snippet: zmk-usb-logging`, or put
   `CONFIG_ZMK_USB_LOGGING=y` in the shield `.conf` (the snippet is used here so the logging
   config lives only in the CI build).
2. Flash, connect the half over USB, and read its ZMK log (e.g. `sudo screen /dev/ttyACM0`).
   Split half stops reporting when the link connects — start it unpaired or watch the start of
   the log.
3. Press one known key (e.g. `S`) and look for a line like:
   ```
   <dbg> zmk: kscan_matrix_read: Sending event at 3,0 state on
   <wrn> zmk: Not found in transform: row: 3, col: 0, pressed: true
   ```
   `(row, col)` here is whatever the current `row-gpios`/`col-gpios` assignment says the key is.
4. Cross-reference that `(row, col)` against your board's actual electrical wiring and fix
   `skeletyl.dtsi` + the half overlays, or rewrite the `default_transform` map so the logical
   keymap positions line up with the physical matrix.
5. Verify `S` now maps to keymap position 11 (`RC(1,1)`), then test the thumb keys — their
   physical order and the transform order should agree (positions 30/31/32 left, 33/34/35 right).

## Building

Push to this repo; GitHub Actions (`build-user-config.yml`) builds both halves. Artifacts:

- `skeletyl_left` — with `zmk-usb-logging` snippet so the central half can be debugged
- `skeletyl_right`

`build.yaml` controls the build matrix. Local build: standard ZMK `west build`
`--board nice_nano//zmk --shield skeletyl_left` (+ `skeletyl_right`).

## Layout

3x5 + 3 thumbs per half. Layers: `QWERTY`, `LOWER` (mo 1), `ADJUST` (mo 2 / mo 3),
`MACRO`, `FUNC`, `BLUETOOTH`. Homing rows use hold-tap (`hm`) for GUI/ALT/SHIFT/CTRL.

Useful references:

- https://docs.bastardkb.com/help/bluetooth.html — BastardKB wireless build guide
- https://github.com/280Zo/charybdis-wireless-mini-zmk-firmware — same matrix rows/cols, 3x6
- https://github.com/s6t/zmk-shield-skeletyl — flex-PCB Skeletyl shield with the same pins