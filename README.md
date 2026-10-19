# local_manifests

The local manifest for building LineageOS 24.0 (Android 17) for the Redmi Note 13 5G (`gold`), and how to build it.

Releases, and what works on them, are in [lineageos-gold](https://github.com/Star-caorui/lineageos-gold).

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0) |
| `device/xiaomi/gold-kernels` | the [DTBO and module lists of the kernel](https://github.com/Star-caorui/android_device_xiaomi_gold-kernels/tree/lineage-24.0); `build-kernel.sh` adds the modules, the device tree blob and the kernel image |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0) |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |
| `lineage-sdk`, `packages/apps/Updater`, `packages/modules/Bluetooth` | LineageOS's, with one change each that is under review on LineageOS's Gerrit ([508379](https://review.lineageos.org/c/LineageOS/android_lineage-sdk/+/508379), [508380](https://review.lineageos.org/c/LineageOS/android_packages_apps_Updater/+/508380), [508408](https://review.lineageos.org/c/LineageOS/android_packages_modules_Bluetooth/+/508408)), from our forks until it is merged |

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs
git clone -b lineage-24.0 https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c

device/xiaomi/gold/build-kernel.sh
source build/envsetup.sh
breakfast gold userdebug
m bacon
```

The output is in `out/target/product/gold/`.

`build-kernel.sh` builds the vendor kernel modules from source and fetches Google's GKI builds, as LineageOS's `build_kernel` does for the devices whose kernel manifests it hosts: it syncs [android_kernel_xiaomi_gold_manifest](https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest#readme) into `out-kernel/gold` and puts the result into `device/xiaomi/gold-kernels`. Run it again after a sync, before building; `SKIP_KERNEL_SYNC=1` builds without syncing the kernel sources first. It needs to reach `android.googlesource.com` and `ci.android.com`.

## Choosing the kernel

Releases use the GKI build stock ships. `GOLD_GKI=latest` picks Google's newest build of the same KMI generation instead, for testing; the vendor modules are the same:

```sh
export GOLD_GKI=latest
source build/envsetup.sh
breakfast gold userdebug
m bacon
```

The DTBO is stock's either way; the DTB is built from source.

## Where the kernel and the proprietary files come from

The kernel image and the GKI modules are Google's builds. The vendor modules and the device tree blob are built from the sources stock's are built from; the [`OS3.0.5.0.VNQMIXM`](https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest/tree/OS3.0.5.0.VNQMIXM) branch of android_kernel_xiaomi_gold_manifest shows they give stock's modules and stock's DTB, byte for byte. The DTBO and the proprietary files are from Xiaomi's stock package; which one and how to check the files against it is in the READMEs of [android_device_xiaomi_gold-kernels](https://github.com/Star-caorui/android_device_xiaomi_gold-kernels/tree/lineage-24.0#readme) and [android_vendor_xiaomi_gold](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0#readme); the second also shows how to make its files yourself.
