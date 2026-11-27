# esphome-mipi-display

## Purpose
MIPI DSI displays and touch controllers: DSI link options, generic command mode, I2C writes in init sequences,
video-only bridges (DSI to HDMI), up to three frame buffers with a scanout interface LVGL draws into, LVGL in RGB888,
touch drivers (GSL3680, EDT-M06, FT5x06) and I2C bus recovery. Display models such as the Raspberry Pi 7" v1 live here.

## How it works
- `mipi_dsi` drives the ESP32-P4's DSI host: the model's init sequence (DCS, generic commands, I2C writes to a bridge
  or backlight controller), then the video stream out of frame buffers in PSRAM; it needs `esp_ldo` for the DSI PHY.
- With 2-3 `frame_buffers` the display scans out of one buffer while LVGL or a video writer draws into another
  (scanout interface); the buffers flip at the frame end.
- `lvgl` renders straight into those frame buffers and can run in RGB888 (`color_depth: 32`).
- The touch drivers (`gsl3680`, `ft5x06`, `ft63x6`) and `i2c` bus recovery work on every chip.
- This repo owns `display`, `lvgl`, `mipi`, `mipi_dsi`, `i2c`, `touchscreen`, `ft5x06`, `ft63x6`, `gsl3680`.

## Quick start
Chips: ESP32-P4
Needs: [esphome-storage-drivers](https://github.com/p1ngb4ck/esphome-storage-drivers) (`esp_ldo`)
```yaml
external_components:
  - source: github://p1ngb4ck/esphome-mipi-display@main
    components: [mipi, mipi_dsi, display, lvgl, i2c, touchscreen, gsl3680]
  - source: github://p1ngb4ck/esphome-storage-drivers@main
    components: [esp_ldo]
esp_ldo:
  - {channel: 3, voltage: 2.5V}
i2c: {sda: GPIO7, scl: GPIO8}
display:
  - platform: mipi_dsi
    id: main_display
    model: WAVESHARE-4.3-DSI
    frame_buffers: 2
touchscreen:
  - {platform: gsl3680, display: main_display, interrupt_pin: GPIO21, reset_pin: GPIO22}
lvgl: {displays: [main_display]}
```
Use your board's pins and your panel's model. Take these components only from here, not from other topic repos.

## Main options

| Option | What it does | Default |
|---|---|---|
| [`frame_buffers`](CONFIGURATION.md#mipi_dsi-display) | frame buffers in PSRAM, 1-3; 2+ = scanout and flip | `1` |
| [`command_mode`](CONFIGURATION.md#mipi_dsi-display) | `dcs`, or `generic` for bridge chips | `dcs` |
| [`video_mode`](CONFIGURATION.md#mipi_dsi-display) | `burst` or a `non_burst_sync_*` mode | `burst` |
| [`video_mode_only`](CONFIGURATION.md#mipi_dsi-display) | no DBI / DCS commands (video-only bridges) | `false` |
| [`continuous_clock`](CONFIGURATION.md#mipi_dsi-display) | DSI clock lane stays in high-speed mode | `false` |
| [`read_registers`](CONFIGURATION.md#mipi_dsi-display) | `generic` mode: registers read back and logged | `[]` |
| [`color_depth`](CONFIGURATION.md#lvgl) | lvgl `32` = RGB888 | `16` |
| [`firmware_reads`](CONFIGURATION.md#touch) | ft5x06: touches as the controller's firmware reports them | `false` |
| [`bus_recovery`](CONFIGURATION.md#touch) | i2c: recover a stuck bus before use | - |

## Catalog
- Components (folders on main; options):
  [mipi](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/mipi),
  [mipi_dsi](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/mipi_dsi),
  [display](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/display)
  ([options](CONFIGURATION.md#mipi_dsi-display));
  [lvgl](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/lvgl)
  ([options](CONFIGURATION.md#lvgl));
  [touchscreen](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/touchscreen),
  [ft5x06](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/ft5x06),
  [ft63x6](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/ft63x6),
  [gsl3680](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/gsl3680),
  [i2c](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main/esphome/components/i2c)
  ([options](CONFIGURATION.md#touch)).
- Branches: [main](https://github.com/p1ngb4ck/esphome-mipi-display/tree/main),
  [pr/ft5x06-firmware-reads](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/ft5x06-firmware-reads),
  [pr/ft5x06-vendor-ids](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/ft5x06-vendor-ids),
  [pr/ft63x6-edt-m06](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/ft63x6-edt-m06),
  [pr/gsl3680](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/gsl3680),
  [pr/i2c-bus-recovery-option](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/i2c-bus-recovery-option),
  [pr/lvgl-color-depth-32](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/lvgl-color-depth-32),
  [pr/mipi-dsi-generic-commands](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-generic-commands),
  [pr/mipi-dsi-link-options](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-link-options),
  [pr/mipi-dsi-read-registers](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-read-registers),
  [pr/mipi-dsi-rpi-7inch-v1](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-rpi-7inch-v1),
  [pr/mipi-dsi-scanout](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-scanout),
  [pr/mipi-dsi-video-mode](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-video-mode),
  [pr/mipi-dsi-video-mode-only](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-dsi-video-mode-only),
  [pr/mipi-i2c-init-writes](https://github.com/p1ngb4ck/esphome-mipi-display/tree/pr/mipi-i2c-init-writes).
- Needed repos: [esphome-storage-drivers](https://github.com/p1ngb4ck/esphome-storage-drivers) (`esp_ldo`).
- All topics together: `github://p1ngb4ck/esphome@dev`.

---

# Detailed version

MIPI DSI displays on the ESP32-P4 and touch controllers on every chip: link options for bridge chips and difficult
panels, generic command mode, I2C writes in init sequences, video-only bridges (DSI to HDMI), up to three frame
buffers with a scanout interface that LVGL and video writers draw into directly, LVGL in RGB888, and touch drivers
(GSL3680, EDT-M06 variants, FT5x06 fixes, I2C bus recovery).

It takes over [esphome-rpi-7inch-display](https://github.com/p1ngb4ck/esphome-rpi-7inch-display) (the Raspberry Pi
7" Touch Display v1, model `RPI-7INCH-V1`) and
[esphome-dfrobot-dfr0550-display](https://github.com/p1ngb4ck/esphome-dfrobot-dfr0550-display) (FT5x06 firmware
reads, I2C bus recovery); both are superseded by this repository. They still exist with their own, diverged copies
of `ft5x06` / `mipi_dsi`, and the DFR0550 display model (`mipi_dsi/models/dfrobot.py`) is not on this `main` yet.

## Examples

A DSI panel with two frame buffers, LVGL drawing straight into them, and a GSL3680 touch controller:

```yaml
external_components:
  - source: github://p1ngb4ck/esphome-mipi-display@main
    components: [mipi, mipi_dsi, display, lvgl, touchscreen, ft5x06, ft63x6, gsl3680, i2c]
  - source: github://p1ngb4ck/esphome-storage-drivers@main
    components: [esp_ldo]

esp_ldo:
  - channel: 3
    voltage: 2.5V

i2c:
  sda: GPIO7
  scl: GPIO8

display:
  - platform: mipi_dsi
    id: main_display
    model: WAVESHARE-4.3-DSI
    frame_buffers: 2

lvgl:
  displays: [main_display]
```

A GSL3680 touch controller:

```yaml
touchscreen:
  - platform: gsl3680
    display: main_display
    interrupt_pin: GPIO21
    reset_pin: GPIO22
```

Use your board's pins and your panel's model. `mipi_dsi` depends on `esp_ldo`
([esphome-storage-drivers](https://github.com/p1ngb4ck/esphome-storage-drivers)) and `psram`.

Needs and conflicts: take `display`, `lvgl`, `mipi`, `mipi_dsi`, `i2c`, `touchscreen`, `ft5x06`, `ft63x6` and
`gsl3680` only from this repository. Other topic repos (esphome-esp-video, esphome-browser, esphome-emulator,
esphome-remote-desktop, esphome-usb-drivers) carry their own copies of display / lvgl / mipi_dsi; do not combine
those copies with this repository - use `github://p1ngb4ck/esphome@dev` for such combinations.

## More

**[CONFIGURATION.md](CONFIGURATION.md)**: every new option - DSI link options, command modes, register read-back,
video-only bridges and their models, frame buffers and scanout, LVGL `color_depth: 32`, the touch options.

## Branches

One branch per change; `main` contains every branch. Base: ESPHome `dev` at `a5a75738ad`.

- `pr/mipi-dsi-link-options` (ESP32-P4): mipi_dsi DSI link options for bridge chips and difficult panels.
- `pr/mipi-dsi-generic-commands` (ESP32-P4): mipi_dsi generic command mode.
- `pr/mipi-dsi-read-registers` (ESP32-P4): mipi_dsi `read_registers`: in generic command mode each listed 16-bit
  register read back after the init sequence and logged.
- `pr/mipi-dsi-video-mode` (ESP32-P4): mipi_dsi video mode.
- `pr/mipi-dsi-video-mode-only` (ESP32-P4): mipi_dsi `video_mode_only` (no DBI / DCS commands; delays and I2C writes
  only), models `ECA20-MHA1-I` (DSI to HDMI) and `WAVESHARE-4.3-DSI`.
- `pr/mipi-i2c-init-writes` (ESP32-P4): mipi: I2C writes in init sequences.
- `pr/mipi-dsi-rpi-7inch-v1` (ESP32-P4): mipi_dsi model `RPI-7INCH-V1`.
- `pr/mipi-dsi-scanout` (ESP32-P4): mipi_dsi `frame_buffers` (up to 3), the frame buffers handed to a direct client
  and flipped at the frame end, the scanout interface, LVGL drawing straight into the frame buffers with two of them.
- `pr/lvgl-color-depth-32` (ESP32-P4): lvgl `color_depth: 32` as RGB888 (3 bytes per pixel).
- `pr/ft63x6-edt-m06` (all): ft63x6, touchscreen: EDT-M06 work-mode reads (`edt_read_variant`).
- `pr/gsl3680` (all): gsl3680 touchscreen driver.
- `pr/ft5x06-firmware-reads` (all): ft5x06 `firmware_reads`.
- `pr/ft5x06-vendor-ids` (all): ft5x06: unknown vendor ids.
- `pr/i2c-bus-recovery-option` (all): i2c `bus_recovery`.

All topics together: `github://p1ngb4ck/esphome@dev`. `external_components` replaces components only; the ESPHome
core of the build host stays as installed.

## License

- ESPHome's own files, also where this repository changes them: the [ESPHome License](LICENSE) (MIT for the Python
  code and everything else, GPLv3 for the C++ runtime files).
- New files written for this repository (new components, their code, tests and documentation): MIT,
  [LICENSE-MIT](LICENSE-MIT).
- Third-party code keeps its own licence:
  - `gsl3680/`: the GSL3680 firmware and driver parts carry Silead's GPL notice.

## Thanks

The GSL3680 driver comes from Konstantin's ([kvj](https://github.com/kvj/esphome)) `jd9365_gsl3680`, which
reached this repository through [NonaSuomy](https://github.com/NonaSuomy)'s esphomebrew - thank you both. Silead's
GPL notice stays in its files. Everyone this project builds on:
[ATTRIBUTIONS.md](https://github.com/p1ngb4ck/esphome/blob/dev/ATTRIBUTIONS.md).

## Branch structure

Each `pr/` branch holds one part of this topic on top of upstream and says in its `BRANCH.md` what it is based on.
`main` holds all of them.

| Branch | Based on |
|---|---|
| `pr/ft5x06-firmware-reads` | upstream only |
| `pr/ft5x06-vendor-ids` | upstream only |
| `pr/ft63x6-edt-m06` | `pr/ft5x06-firmware-reads`, `pr/ft5x06-vendor-ids`, `pr/i2c-bus-recovery-option` |
| `pr/gsl3680` | `pr/ft5x06-firmware-reads`, `pr/ft5x06-vendor-ids`, `pr/i2c-bus-recovery-option` |
| `pr/i2c-bus-recovery-option` | upstream only |
| `pr/lvgl-color-depth-32` | `pr/ft5x06-firmware-reads`, `pr/ft5x06-vendor-ids`, `pr/i2c-bus-recovery-option` |
| `pr/mipi-dsi-generic-commands` | upstream only |
| `pr/mipi-dsi-link-options` | upstream only |
| `pr/mipi-dsi-read-registers` | `pr/ft5x06-firmware-reads`, `pr/ft5x06-vendor-ids`, `pr/i2c-bus-recovery-option` |
| `pr/mipi-dsi-rpi-7inch-v1` | upstream only |
| `pr/mipi-dsi-scanout` | `pr/ft5x06-firmware-reads`, `pr/ft5x06-vendor-ids`, `pr/i2c-bus-recovery-option` |
| `pr/mipi-dsi-video-mode` | `pr/ft5x06-vendor-ids` |
| `pr/mipi-dsi-video-mode-only` | `pr/ft5x06-firmware-reads`, `pr/ft5x06-vendor-ids`, `pr/i2c-bus-recovery-option` |
| `pr/mipi-i2c-init-writes` | upstream only |

`pr/gsl3680`, `pr/ft63x6-edt-m06` and `pr/lvgl-color-depth-32` also carry `mipi` / `mipi_dsi` and `web_server`
changes that are not their own change; they are to be rebased on upstream only.
