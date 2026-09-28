# Cirque Pinnacle driver for ZMK (absolute mode)

Zephyr's in-tree `input_pinnacle` driver for Cirque Pinnacle trackpads,
packaged as a Zephyr module so it builds against ZMK 0.3 / Zephyr 3.5
trees, which predate it. It reports absolute position and touch
strength (`data-mode = "absolute"`), which the
[`zmk-raw-touch`](https://github.com/kalakris/zmk-raw-touch) module
needs.

Use the `intree-driver` branch, pinned to a release tag (or `stable`),
as the `zmk-raw-touch` README describes. Other branches in this
repository are history from the fork it started as.

## What is here

- `drivers/input/input_pinnacle.c`: the upstream Zephyr driver, vendored
  verbatim from `zephyrproject-rtos/zephyr` (commit `27150c9d`), with two
  build shims for Zephyr 3.5 and three small patches on top:
  1. Ignore suspicious `0xFF` status reads and frames without new data.
  2. Per-axis edge sensitivity (`x-axis-z-min`, `y-axis-z-min`).
  3. A recalibration at the end of initialization.
- `dts/bindings/`: the upstream devicetree bindings, plus the two
  properties above.

The file header and the branch history list every local change.

## Credits

The driver is Zephyr's, by the Zephyr contributors. The three patches
re-apply fixes and features from Pete Johanson's
[cirque-input-module](https://github.com/petejohanson/cirque-input-module),
the Pinnacle driver fork this repository was forked from, rewritten
against the upstream driver.

## License

[Apache-2.0](LICENSE), the license of the upstream Zephyr driver.
