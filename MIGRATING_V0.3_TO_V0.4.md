# Migrating from ZMK v0.3 to v0.4 (fingerpunch)

ZMK has moved from Zephyr 3.5 to Zephyr 4.1, introducing Hardware Model v2 (HWMv2). This guide covers what **you** need to change in your own `zmk-config` repo if you followed the `zmk-config-fp-example` template.

All fingerpunch upstream repos (`zmk-fingerpunch-keyboards`, `zmk-fingerpunch-controllers`, `zmk-fingerpunch-vik`) have already been updated. You only need to update your own config.

---

## Quick Start: Who Needs to Do What?

| Your Setup | What to Update |
|---|---|
| Standard keyboard, no Cirque, no custom overlays | `west.yml` + `build.yaml` + workflow |
| Keyboard with VIK Cirque module | `west.yml` + `build.yaml` + workflow |
| Keyboard with directly-wired Cirque trackpad | `west.yml` + `build.yaml` + workflow + Cirque overlay properties |
| Any of the above with custom overlay referencing old board names | Also rename per-board overlay files (see Step 4) |

---

## Step 1: Update `config/west.yml`

Replace the contents of your `config/west.yml` with:

```yaml
manifest:
  remotes:
    - name: zmkfirmware
      url-base: https://github.com/zmkfirmware
    - name: sadekbaroudi
      url-base: https://github.com/sadekbaroudi
  projects:
    - name: zmk
      remote: zmkfirmware
      revision: main
      import: app/west.yml
    - name: zmk-fingerpunch-keyboards
      remote: sadekbaroudi
      revision: main
      import: config/deps.yml
  self:
    path: config
```

**What changed:**
- The `zmk` project now points to `zmkfirmware/zmk@main` instead of `petejohanson/zmk@feat/pointers-move-scroll`
- Remove the `petejohanson` remote entirely
- Remove `cirque-input-module` if you had it — the Cirque Pinnacle driver is now built into Zephyr

---

## Step 2: Update `build.yaml`

Board names have changed. Update your `build.yaml`:

| Old Board Name | New Board Name |
|---|---|
| `nice_nano_v2` | `nice_nano` |
| `nice_nano` (v1) | `nice_nano@1` |
| `seeeduino_xiao_ble` | `xiao_ble` |
| `seeeduino_xiao_rp2040` | `xiao_rp2040` |

> **Note:** `seeeduino_xiao` (original SAMD21 XIAO) is unchanged.

> **Note:** Fingerpunch board names (`vikoto`, `svlinky`, `xivik`, `ffkb_holyiot_v1`, `pinkies_out_v3`, etc.) are unchanged.

**Example:**
```yaml
# OLD:
include:
  - board: nice_nano_v2
    shield: ffkb_v2

# NEW:
include:
  - board: nice_nano
    shield: ffkb_v2
```

---

## Step 3: Update GitHub Actions Workflow

In `.github/workflows/build.yml`, make sure you're pointing to `@main`:

```yaml
on: [push, pull_request, workflow_dispatch]

jobs:
  build:
    uses: zmkfirmware/zmk/.github/workflows/build-user-config.yml@main
```

---

## Step 4: Rename Per-Board Overlay/Conf Files (if you have them)

If you have per-board overlay or config files in your `config/` directory (e.g., `nice_nano_v2.conf`), rename them to match the new board names:

| Old Filename | New Filename |
|---|---|
| `nice_nano_v2.overlay` | `nice_nano.overlay` |
| `nice_nano_v2.conf` | `nice_nano.conf` |
| `nice_nano_v2.keymap` | `nice_nano.keymap` |
| `seeeduino_xiao_ble.conf` | `xiao_ble.conf` |
| `seeeduino_xiao_rp2040.conf` | `xiao_rp2040.conf` |

If you don't have any per-board files, skip this step.

---

## Step 5: Update Cirque Trackpad Overlay (if directly wired)

> **If you use a VIK Cirque module**, the VIK shield handles these changes for you — skip this step.

If you have a directly-wired Cirque trackpad with a custom overlay (like the `ffkb_v2.overlay` example), update these properties:

| Old Property | New Property |
|---|---|
| `dr-gpios` | `data-ready-gpios` |
| `no-taps` | *(remove entirely)* |
| `sleep` | `sleep-mode-enable` |
| `x-invert` / `y-invert` (on Cirque node) | `invert-x` / `invert-y` |
| `rotate-90` (on Cirque node) | `swap-xy` |

> **Important:** The `xy-swap`, `y-invert`, and `x-invert` properties on the `zmk,input-listener` node are **unchanged** — those are ZMK properties, not Cirque driver properties.

> **Note:** `compatible = "cirque,pinnacle"` is still correct — no change needed.

**Example (before):**
```dts
glidepoint: glidepoint@2a {
    compatible = "cirque,pinnacle";
    reg = <0x2a>;
    status = "okay";
    dr-gpios = <&gpio0 6 (GPIO_ACTIVE_HIGH)>;
    sensitivity = "4x";
    sleep;
    no-taps;
};
```

**Example (after):**
```dts
glidepoint: glidepoint@2a {
    compatible = "cirque,pinnacle";
    reg = <0x2a>;
    status = "okay";
    data-ready-gpios = <&gpio0 6 (GPIO_ACTIVE_HIGH)>;
    sensitivity = "4x";
    sleep-mode-enable;
};
```

Note that `no-taps` was removed (not renamed). The upstream driver defaults to taps disabled. If you want taps, add `primary-tap-enable;` instead.

---

## Common Errors

### `Aborting due to Kconfig warnings`
If you see an error about undefined Kconfig symbols, you likely have `CONFIG_WS2812_STRIP=y` in a `.conf` file. Change it to `CONFIG_WS2812_STRIP_SPI=y`.

### `fatal error: cirque-input-module not found` or west manifest errors
Remove `cirque-input-module` from your `config/west.yml`. The Cirque Pinnacle driver is now part of Zephyr.

### Build fails with `nice_nano_v2` not found
The board was renamed to `nice_nano` (v2 is now the default revision). Update your `build.yaml` and any per-board overlay filenames.

### Build fails with `seeeduino_xiao_ble` not found
The board was renamed to `xiao_ble`. Update your `build.yaml` and any per-board overlay filenames.

### Linker errors: `undefined reference to retention_read/retention_write`
This is a board-level issue (already fixed in the fingerpunch repos). If you see this on a custom board, add `imply RETAINED_MEM`, `imply RETENTION`, and `imply RETENTION_BOOT_MODE` to your board's Kconfig file.

---

## Reference

- [ZMK Zephyr 4.1 Migration Blog Post](https://zmk.dev/blog/2025/12/09/zephyr-4-1)
- [zmk-config-fp-example](https://github.com/sadekbaroudi/zmk-config-fp-example) (updated template)
- [zmk-fingerpunch-keyboards](https://github.com/sadekbaroudi/zmk-fingerpunch-keyboards)
