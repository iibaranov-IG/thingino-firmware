# Broadcast Control Lab — Thingino fork

This is Igor Baranov's working fork of Thingino. BCL repair reports are independent engineering results, not official Thingino releases or hardware certification. Original project documentation is retained below.

## Wyze Video Doorbell v2: button events for Home Assistant

**Published: 17 September 2026. Status: BCL host regression PASS; firmware build and hardware acceptance pending.**

The original [Thingino issue #1623](https://github.com/themactep/thingino-firmware/issues/1623), reported by WLTB-Gino, describes a real button that plays the local chime but never sends its Home Assistant doorbell event. Credit for the original diagnosis and on-device workaround belongs to that report. Its successful workaround is not hardware verification of this candidate.

### Existing repair — two profile files only

[Inspect the tested repair commit](https://github.com/iibaranov-IG/thingino-firmware/commit/bf8aea31c6006ca0b41e2b56c486fd5bd8628968) · [Source patch](https://github.com/iibaranov-IG/thingino-firmware/commit/bf8aea31c6006ca0b41e2b56c486fd5bd8628968.patch) · [Existing BCL PR #38](https://github.com/iibaranov-IG/broadcast-control-lab/pull/38)

For profile `wyze_vdb2_t31x_sc301iot_atbm6031`, the patch enables `BR2_PACKAGE_WYZE_ACCESSORY` and `BR2_PACKAGE_WYZE_ACCESSORY_DOORBELL_CTRL`, and sets `chime.bypass=true`. Existing package hooks then install the event handler and its `KEY_1 RELEASE` rule. The original `KEY_1 TIMED` rule retains responsibility for the local sound; bypass avoids requiring a VDB1 wireless-chime sound mapping.

No shared scripts, GPIO assignments, reset configuration, networking, bootloader or other camera profiles are changed by the repair.

| Identity | Exact revision |
| --- | --- |
| Upstream baseline | `4b48cff9d1ff9f6c32d8bdbffeb76a4695c22f45` |
| Tested repair in this fork | `bf8aea31c6006ca0b41e2b56c486fd5bd8628968` |
| Candidate source tree | `4636499e454fccdc109c8a5fb7c358bc90698572` |
| Existing repair branch | `bcl/thingino-1623-vdb2-doorbell` |
| BCL test revision | `51257d03313052b9ad6a35eb4e88959d5155706c` |

**Use the exact repair commit above to identify the tested source. This README publication does not merge the repair into this fork's `master`, and a later branch tip is not automatically covered by this evidence.**

### Completed BCL red → green verification

[BCL v2 run #203 — completed successfully](https://github.com/iibaranov-IG/broadcast-control-lab/actions/runs/35200485849) · [Evidence archive](https://github.com/iibaranov-IG/broadcast-control-lab/actions/runs/35200485849/artifacts/10487382760)

| Stage | Recorded result |
| --- | --- |
| Unpatched baseline, unchanged 11-test regression | Expected failure: exit 1; 3 passed, 8 failed, 0 skipped |
| Patched candidate, same regression | Exit 0; 11 passed, 0 failed, 0 skipped |
| Candidate suite, same configured 11-test scope | Exit 0; 11 passed, 0 failed, 0 skipped |

The checks execute selected production GNU make install/finalize recipes and filesystem-rebased production `doorbell_event` / `ha-common` shell logic. They cover package selection, executable handler installation, one release rule and one local-chime rule, captured retained `ON` publications in day/night modes, repeated presses with cached `ON`, disabled or absent Home Assistant support, preserved GPIO/reset configuration, and shell syntax.

`jct`, GPIO/LED, sleep, playback and `mosquitto_pub` are explicit test shims. Captured MQTT command arguments are not delivery through a live broker. Preserved playback commands are not a measurement of actual sound. The configured candidate suite is the focused 11-test regression, not the full Thingino test suite.

Reproduce from the recorded BCL revision with its documented runtime prerequisites:

```sh
git checkout --detach 51257d03313052b9ad6a35eb4e88959d5155706c
node scripts/bcl.cjs run thingino-firmware-1623
```

Run these commands in a separate checkout of `iibaranov-IG/broadcast-control-lab`, not in this firmware repository. [Pinned case and test sources](https://github.com/iibaranov-IG/broadcast-control-lab/tree/51257d03313052b9ad6a35eb4e88959d5155706c/cases/thingino-firmware-1623).

Evidence archive: `thingino-firmware-1623-evidence.zip`, artifact ID `10487382760` (14 files). SHA-256: `c7a972c76c6167a5e0d82ad649e5113c19fccc6ffa004b0d3ede21e78acd759c`. The archive checksum and recorded log/report hashes were checked before this publication. The archive contains the machine-readable evidence, candidate manifest, baseline/candidate logs and owner-check instructions. GitHub artifact retention is finite; the archive checksum identifies the saved copy.

### Remaining verification

No firmware image was built or flashed in this BCL run. Physical GPIO timing/debounce, actual sound, live MQTT delivery, the subsequent `OFF` transition, Home Assistant UI behavior, reboot, video and firmware image footprint still need verification on the exact candidate image. Hardware and full-application verification remain **false**.

The existing case records manual connector-assisted selection review, not output from the `bcl triage` CLI. This fork-local publication does not claim upstream approval or completion of additional upstream-publication gates. Discussion and evidence stay in the existing [BCL PR #38](https://github.com/iibaranov-IG/broadcast-control-lab/pull/38); no duplicate upstream PR is created by this publication.

---

## Original upstream documentation

Thingino
--------

Thingino (_/θinˈdʒiːno/_, _thin-jee-no_) is an open-source firmware for Ingenic SoC IP cameras.

![Thingino Web UI][10]

### Supported Hardware

Please find [the full list of supported cameras](docs/hardware/supported-hardware.md)
in a separate document. Visit [our website][0] for an illustrated version of
the list.

---

### Thingino Repository Branches Explaned

We've split the Thingino repository into two branches: stable and master, to better manage development and provide reliable releases for users.

**Ciao Branch**

Provides a reliable, tested version of Thingino for general use. It includes carefully selected, stable changes. It uses the original ONVIF server and Prudynt with libconfig.
The ciao branch will receive critical fixes. New features will only be added once they are thoroughly tested and mature in the master branch.

For users who want a dependable version of Thingino without needing to build or contribute to development.

**Master Branch**

The development hub for new features and experimental changes. Includes advanced features, and uses the new [raptor][13] streamer. These are still in development and may not be stable.

Only for developers and contributors who can build the project themselves and actively participate in improving the code.

> [!WARNING]
> The master branch uses a highly experimental maineline U-Boot.
> - Having access to the UART port on the camera and unbricking skills is **highly recommended** when building images from the master branch.

This structure allows us to maintain a reliable version (stable) for most users while continuing to innovate and test new features (master). Critical fixes and matured features from master will be gradually integrated into stable for broader use.

> [!NOTE]
> If you’re not contributing to development, we recommend sticking with the stable branch.

Thank you for using Thingino! For questions or contributions, please join our Discord community or check the GitHub issues page.

### Building

```
git clone -b stable --recurse-submodules https://github.com/themactep/thingino-firmware
cd thingino-firmware
make update
make
```

Read [Building from sources][7] article for more info.

### Building in a Container

```
git clone -b stable --recurse-submodules https://github.com/themactep/thingino-firmware
cd thingino-firmware
./build-container.sh
```

The build uses a prebuilt image from
[ghcr.io/themactep/thingino-builder-image](https://github.com/themactep/thingino-builder-image),
pulled automatically on first run. Requires Podman or Docker.

Read [Building in a container](docs/build/container.md) for more info.

### Documentation

- [Firmware Image Structure](docs/firmware/firmware-image-structure.md) - Partition layout and image assembly
- [Firmware Dumping](docs/firmware/firmware.md) - How to backup existing firmware
- [Camera Recovery](docs/firmware/camera-recovery.md) - Recovering from failed updates
- [Local Build Settings](docs/build/local-build-settings.md) - Layered user-specific settings from `THINGINO_USER_DIR/common`, per camera, and per device IP

### Resources

- [Project Website][0]
- [Project Wiki][1]
- Buildroot Manual [HTML][5] [PDF][6]
- [Discord channel][3]
- [Telegram group][4]

### GitHub CI Status

[![toolchain-x86_64][11]][8]
[![firmware-stable][12]][9]

[0]: https://thingino.com/
[1]: https://github.com/themactep/thingino-firmware/wiki
[3]: https://discord.gg/xDmqS944zr
[4]: https://t.me/thingino
[5]: https://buildroot.org/downloads/manual/manual.html
[6]: https://nightly.buildroot.org/manual.pdf
[7]: https://github.com/themactep/thingino-firmware/wiki/Building-from-sources
[8]: https://github.com/themactep/thingino-firmware/actions/workflows/toolchain.yaml
[9]: https://github.com/themactep/thingino-firmware/actions/workflows/firmware.yaml
[10]: https://github.com/user-attachments/assets/5e74827c-47f9-4ea0-b523-d12a199a9974
[11]: https://github.com/themactep/thingino-firmware/actions/workflows/toolchain-x86_64.yaml/badge.svg
[12]: https://github.com/themactep/thingino-firmware/actions/workflows/firmware-stable.yml/badge.svg
[13]: http://github.com/gtxaspec/raptor
