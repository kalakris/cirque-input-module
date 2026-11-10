# Cirque Pinnacle driver for ZMK (absolute mode)

Zephyr's in-tree `input_pinnacle` driver for Cirque Pinnacle trackpads,
packaged as a Zephyr module so it builds against ZMK 0.3 / Zephyr 3.5
trees, which predate it. It can report absolute position and touch
strength (`data-mode = "absolute"`), which the
[`zmk-raw-touch`](https://github.com/kalakris/zmk-raw-touch) module
needs.

## Which repository?

| You have | Start here |
|---|---|
| A MoErgo Go60 | [go60-rawtouch-config](https://github.com/kalakris/go60-rawtouch-config): prebuilt firmware, or a configuration to fork |
| Another ZMK keyboard with a Cirque Pinnacle trackpad | [zmk-raw-touch](https://github.com/kalakris/zmk-raw-touch): the firmware module and its setup guide |
| Firmware with the module, and a Mac | [RawTouch](https://github.com/kalakris/rawtouch): the macOS app |
| A question about the trackpad driver | [cirque-input-module](https://github.com/kalakris/cirque-input-module) (this repository): the absolute-mode Pinnacle driver the module is tested with |

## Using it

This module is for Zephyr 3.5-era trees such as ZMK 0.3 and MoErgo's
ZMK. Newer Zephyr releases include the driver themselves.

Add it to your `config/west.yml`, pinned to a release tag:

```yaml
manifest:
  remotes:
    - name: kalakris
      url-base: https://github.com/kalakris
  projects:
    - name: cirque-input-module
      remote: kalakris
      revision: v0.1.0
```

The project must be named `cirque-input-module`. Some ZMK trees,
MoErgo's among them, already import a project of that name with Pete
Johanson's driver. West keeps the first definition of a name, and your
manifest's own projects come before its imports, so this entry replaces
the imported one. If your `west.yml` already lists a
`cirque-input-module` project, replace that entry; two projects with one
name fail. The moving tag `stable` follows the newest release, if you
would rather not edit the tag for each one.

The devicetree properties differ from Pete Johanson's
[cirque-input-module](https://github.com/petejohanson/cirque-input-module).
Delete the old ones from your board's pad node with `/delete-property/`
and set their replacements:

| petejohanson/cirque-input-module | This driver |
|---|---|
| `dr-gpios` | `data-ready-gpios` |
| `rotate-90` | `swap-xy` (relative mode only); with `zmk-raw-touch`, set `rotate-90` on its pad node instead |
| `x-invert`, `y-invert` | `invert-x`, `invert-y`; with `zmk-raw-touch`, set `x-invert` / `y-invert` on its pad node instead |
| `sleep` | `sleep-mode-enable` |
| Taps (on unless `no-taps`) | `primary-tap-enable`, relative mode only; taps are off without it. Absolute mode has no hardware taps; `zmk-raw-touch`'s `tap-click` replaces them |
| `no-taps`, `no-secondary-tap` | Delete; secondary taps are always off |
| `sensitivity` | `sensitivity`; the default here is `4x`, where it was `1x` |
| `x-axis-z-min`, `y-axis-z-min` | Unchanged |
| `CONFIG_ZMK_INPUT_PINNACLE_IDLE_SLEEPER` | No equivalent; remove it from your `.conf` |

A minimal absolute-mode pad on SPI:

```dts
#include <zephyr/dt-bindings/gpio/gpio.h>

&spi0 {
    status = "okay";
    cs-gpios = <&gpio0 17 GPIO_ACTIVE_LOW>;

    trackpad: trackpad@0 {
        compatible = "cirque,pinnacle";
        reg = <0>;
        spi-max-frequency = <1000000>;
        data-ready-gpios = <&gpio0 4 (GPIO_ACTIVE_HIGH | GPIO_PULL_DOWN)>;
        sensitivity = "1x";
        data-mode = "absolute";
        idle-packets-count = <3>;
    };
};
```

Use your board's bus, chip select and data-ready pin. The pad also works
on I²C with the same `cirque,pinnacle` compatible. In absolute mode,
`idle-packets-count` must be above zero for the pad to report lift-off.
The [bindings](dts/bindings/input/cirque,pinnacle-common.yaml) list every
property. The `zmk-raw-touch`
[setup guide](https://github.com/kalakris/zmk-raw-touch#setup) shows the
rest of a working configuration.

## What is here

- `drivers/input/input_pinnacle.c`: the upstream Zephyr driver, vendored
  verbatim from `zephyrproject-rtos/zephyr` (commit `27150c9d`), with two
  build shims for Zephyr 3.5 and six small patches on top:
  1. A status read of `0xFF`, or a frame without new data, is dropped
     rather than reported as motion.
  2. Per-axis edge sensitivity (`x-axis-z-min`, `y-axis-z-min`).
  3. The pad recalibrates at the end of initialization, at the
     configured sensitivity.
  4. A dropped frame never leaves the data-ready line stuck high, so the
     pad keeps reporting.
  5. The command-complete flag is cleared only after it has latched, and
     a flag that latches late does not fail initialization.
  6. `x-axis-z-min` and `y-axis-z-min` outside 0–255 fail the build
     instead of being truncated.
- `dts/bindings/`: the upstream devicetree bindings, plus the two
  properties in patch 2.

The file header and the commit history list every local change.

## Known limitations

- Rarely, a pad does not respond after power-on while the keys work.
  Switch that half off and on again.

## Credits

The driver is Zephyr's, by the Zephyr contributors. Patches 1 to 3
apply fixes and features from Pete Johanson's
[cirque-input-module](https://github.com/petejohanson/cirque-input-module)
to the upstream driver.

## License

[Apache-2.0](LICENSE), the license of the upstream Zephyr driver, except
where a file's header says otherwise.
