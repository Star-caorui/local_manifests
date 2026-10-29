# local_manifests

The local manifest for building LineageOS 24.0 (Android 17) for the Redmi Note 13 5G (`gold`), and how to build it.

Releases, and what works on them, are in [lineageos-gold](https://github.com/Star-caorui/lineageos-gold).

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0) |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0) |
| `device/xiaomi/gold-kernel` | the kernel, prebuilt: a tag of [android_kernel_xiaomi_gold](https://github.com/Star-caorui/android_kernel_xiaomi_gold/tree/release)'s `release` branch |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |

## Patches

`gold.xml` replaces none of LineageOS's projects. The build changes a few of them with the device tree's [platform patches](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0#platform-patches), which `device/xiaomi/gold/patches/apply.sh lineage` commits after every `repo sync`, as below.

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs
git clone -b lineage-24.0 https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c
device/xiaomi/gold/patches/apply.sh lineage

source build/envsetup.sh
breakfast gold userdebug
m bacon
```

The output is in `out/target/product/gold/`.

## The kernel

The build takes the kernel prebuilt from `device/xiaomi/gold-kernel`: android_kernel_xiaomi_gold's Release workflow
builds it with [android_kernel_xiaomi_gold_manifest](https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest/tree/lineage-24.0#readme)
and commits it to its `release` branch, one tag per build, laid out as LineageOS's `build_kernel` lays it out. To use
another build, change the tag in `gold.xml`. To build the kernel from source instead, follow that manifest's README
and point the device tree's `TARGET_KERNEL_DIR` at its output.

## Where the kernel and the proprietary files come from

The kernel image, its certified boot image and the GKI modules are Google's builds. The vendor modules, the device tree blob and its overlay are built from source; [`verify/`](https://github.com/Star-caorui/android_kernel_xiaomi_gold/tree/OS3.0.5.0.VNQMIXM/verify) on android_kernel_xiaomi_gold's `OS3.0.5.0.VNQMIXM` branch shows they give stock's modules, DTB and DTBO. The proprietary files are from Xiaomi's stock package; which one, how to check the files against it and how to make them yourself is in the README of [android_vendor_xiaomi_gold](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0#readme).
