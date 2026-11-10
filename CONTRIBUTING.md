# Contributing to cirque-input-module

Issues and pull requests are welcome. Include your board, ZMK tree,
the bus (SPI or I²C), the pad's devicetree node, and the driver's log
output if you have it.

## Changes

- `drivers/input/input_pinnacle.c` is Zephyr's driver plus a short list
  of patches. Keep each local change its own commit, and mention it in
  the file header.
- Keep the branch building for ZMK 0.3 / Zephyr 3.5. The
  [`zmk-raw-touch`](https://github.com/kalakris/zmk-raw-touch) CI builds
  this driver on a MoErgo Go60; say in the pull request how you built and
  tested a change.

## Releases

Releases are tags on the `intree-driver` branch, `v0.y.z`. The
maintainer tags a tested commit and pushes the tag, then moves the
`stable` tag to it:

```sh
git tag v0.1.0 && git push origin v0.1.0
git tag -f stable v0.1.0 && git push -f origin stable
```

`zmk-raw-touch` release notes name the driver tag each module release
was tested with.
