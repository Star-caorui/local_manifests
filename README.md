# local_manifests

The local manifest for building PixelOS 17 (Android 17) for the Redmi Note 13 5G (`gold`), and how to build it. The `lineage-24.0` branch is the one for LineageOS 24.0.

PixelOS 17 builds with LineageOS 24.0's `vendor/lineage`, so the device tree, the proprietary files and the kernel are the same as for LineageOS.

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0) |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0) |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |

## Patches

`gold.xml` replaces none of PixelOS's projects. The build changes a few of them with the device tree's [platform patches](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0#platform-patches), which `device/xiaomi/gold/patches/apply.sh pixelos` commits after every `repo sync`, as below.

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/PixelOS-AOSP/android_manifest.git -b seventeen --git-lfs
git clone -b seventeen https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c
device/xiaomi/gold/patches/apply.sh pixelos

mkdir -p out-kernel/xiaomi_gold/.repo && touch out-kernel/xiaomi_gold/.out-dir
cp -R .repo/repo out-kernel/xiaomi_gold/.repo/repo
(cd out-kernel/xiaomi_gold &&
  repo init -u https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest -b lineage-24.0 --depth=1 --no-use-superproject &&
  repo sync -c --no-tags)

export SKIP_KERNEL_SYNC=1
source build/envsetup.sh
breakfast gold userdebug
m pixelos
```

The output is in `out/target/product/gold/`. How the kernel is built, and how to choose another GKI build, is in the `lineage-24.0` branch's [README](https://github.com/Star-caorui/local_manifests/tree/lineage-24.0#readme).
