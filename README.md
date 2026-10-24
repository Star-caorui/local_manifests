# local_manifests

The local manifest for building PixelOS 17 (Android 17) for the Redmi Note 13 5G (`gold`), and how to build it. The `lineage-24.0` branch is the one for LineageOS 24.0.

PixelOS 17 builds with LineageOS 24.0's `vendor/lineage`, so the device tree, the proprietary files and the kernel are the same as for LineageOS.

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0) |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0) |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |
| `packages/modules/Bluetooth` | PixelOS's, with the change under review on LineageOS's Gerrit as [508408](https://review.lineageos.org/c/LineageOS/android_packages_modules_Bluetooth/+/508408), until it is merged |
| `system/fs/fs_mgr` | PixelOS's, with Mashopy's fenrir LK compatibility patch from Nothing-2A: [libfs_avb](https://github.com/Nothing-2A/android_system_core/commit/8ff6e7a68523c3b870d8dcd5713c71ea15b43dd2) treats both `orange` and `green` verified boot states as unlocked. PixelOS's fastbootd already reports a device as unlocked when the kernel command line has `fenrir=true` |

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/PixelOS-AOSP/android_manifest.git -b seventeen --git-lfs
git clone -b seventeen https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c

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
