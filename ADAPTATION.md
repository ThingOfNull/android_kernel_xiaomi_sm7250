# kernel_xiaomi_sm7250 — 4.19 vendor kernel, Android 16 (LineageOS 23.2) adaptation

This is the Xiaomi **SM7250 (Snapdragon 765G)** 4.19.325 vendor kernel, adapted
to boot **LineageOS 23.2 (Android 16)** on **vangogh** (Xiaomi Mi 10 Lite Zoom).

## Branch

- `lineage-23.2-4.19` — Android 16 (LineageOS 23.2) + 4.19 vendor kernel.
- `main` mirrors it.

Related repositories:

- [device_xiaomi_vangogh](https://github.com/ThingOfNull/device_xiaomi_vangogh)
- [device_xiaomi_sm7250-common](https://github.com/ThingOfNull/device_xiaomi_sm7250-common)

## Kernel configuration

`arch/arm64/configs/vangogh_419_defconfig` is the full 4.19.325 configuration
extracted from the working boot image (IKCONFIG). It produces a ~46 MB Image and
matches the vendor configuration (WALT, cgroups, audit, etc.). The upstream
`milito_defconfig` in the tree is an experimental minimal config and must not be
used for this device.

## Changes for this adaptation (see the commits)

- **Display (techpack)**: AOD/doze brightness translation for the MI panel
  (HBM/LBM + default LBM on LP1/LP2) and the LiveDisplay HBM sysfs.
- **Touch (FocalTech)**: gate 0x24 gesture reports behind
  `fts_gesture_is_enabled()` so double-tap does not leak into normal touch.
- **Goodix fingerprint (goodix_ta)**: namespace/build fixes for the 4.19 tree.
- **Power (smb5)**: super-charger macro fixes.
- **soc/qcom (service-locator)**: default `present` behaviour for the ADSP
  service locator.
- **WLAN (qcacld)**: TSO stats lock initialisation fix.
- **Core (lockdep / RCU trace)**: boot-time fix for lockdep with
  `DEBUG_LOCK_ALLOC`.
- **Audio (tfa98xx)**: forward-declaration build fix.
- **DTS (lito)**: RCU bootargs / IR SPI binding fixes.

The device tree side (LiveDisplay, doze settings, vendor blobs) lives in the
device repositories.

## Build

The kernel is normally built by the LineageOS build system from the device
tree. Standalone:

```bash
export ARCH=arm64
export CROSS_COMPILE=aarch64-linux-gnu-
make O=out LLVM=1 LLVM_IAS=1 CC=clang vangogh_419_defconfig
make O=out LLVM=1 LLVM_IAS=1 CC=clang -j$(nproc) Image
```

Toolchain: Android clang r563880c (or a matching clang/LLD 21).

## Notes

- The boot image uses header v2 with the combined DTB (production parts + the
  vangogh DTB); the device tree packaging rules live in the device repository.
- Recovery/kernel bring-up switches (`vangogh.*`) are optional and only used by
  development images.

## Status / known issues

Working: the 4.19.325 image boots Android 16, with display/AOD, touch,
fingerprint, Wi-Fi, audio, charging and OTA updates.

Known limitations:

- Vendor 4.19.325 tree; no newer stable/upstream merges.
- `vangogh_419_defconfig` is the known-good configuration extracted from the
  working boot image; keep it in sync with the userspace/HAL expectations.
- The FOD z-order plane extension is supported and required by userspace
  (`TARGET_USES_FOD_ZPOS=true` in the device tree).
- `vangogh.*` command-line switches used during bring-up are development-only.
- The 5.10 kernel port (experimental) is maintained separately and is not part
  of this branch.
