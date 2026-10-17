# local_manifests

The local manifest for building LineageOS 24.0 (Android 17) for the Redmi Note 13 5G (`gold`), and how to build it.

Releases, and what works on them, are in [lineageos-gold](https://github.com/Star-caorui/lineageos-gold).

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0) |
| `device/xiaomi/gold-kernel` | the [stock kernel and its modules](https://github.com/Star-caorui/android_device_xiaomi_gold-kernel/tree/lineage-24.0), prebuilt, and Google's newest GKI build |
| `kernel/xiaomi/gold` | the [Android common kernel](https://github.com/Star-caorui/android_kernel_xiaomi_gold/tree/lineage-24.0) at the commit of the stock GKI image; `gold-kernel-source` group only |
| `kernel/xiaomi/gold-modules` | [MediaTek's device modules](https://github.com/Star-caorui/android_kernel_xiaomi_gold-modules/tree/lineage-24.0), from Xiaomi's release, cut down to what gold builds; `gold-kernel-source` group only |
| `kernel/xiaomi/gold-mediatek_modules` | [MediaTek's connectivity, GPU, fpsgo, MET and scheduler modules](https://github.com/Star-caorui/android_kernel_xiaomi_gold-mediatek_modules/tree/lineage-24.0), from Xiaomi's release, cut down to what gold builds; `gold-kernel-source` group only |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0) |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |
| `lineage-sdk`, `packages/apps/Updater`, `packages/modules/Bluetooth` | LineageOS's, with one change each that is under review on LineageOS's Gerrit ([508379](https://review.lineageos.org/c/LineageOS/android_lineage-sdk/+/508379), [508380](https://review.lineageos.org/c/LineageOS/android_packages_apps_Updater/+/508380), [508408](https://review.lineageos.org/c/LineageOS/android_packages_modules_Bluetooth/+/508408)), from our forks until it is merged |

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs
git clone -b lineage-24.0 https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c

source build/envsetup.sh
breakfast gold userdebug
m bacon
```

The output is in `out/target/product/gold/`.

## Choosing the kernel

Releases use the stock kernel and modules. Two variables pick others, for testing; both default to `stock`:

| Variable | Picks | Values |
| --- | --- | --- |
| `GOLD_GKI` | the kernel image and the GKI modules in `system_dlkm` | `stock`: Google's GKI build that stock ships. `latest`: Google's newest build of the same KMI generation. `source`: built from source |
| `GOLD_VENDOR_MODULES` | the modules in `vendor_boot` and `vendor_dlkm` | `stock`: Xiaomi's and MediaTek's. `source`: built from source |

`GOLD_GKI=source` needs `GOLD_VENDOR_MODULES=source`: stock vendor modules only load on Google's kernel. Anything from source needs the `gold-kernel-source` group:

```sh
repo init -g default,gold-kernel-source
repo sync -c

export GOLD_GKI=latest GOLD_VENDOR_MODULES=source
source build/envsetup.sh
breakfast gold userdebug
m bacon
```

Run `m installclean` when changing either, or modules of the other setting stay in the output.

Stock modules still stand in for the drivers that are not built from source yet; the device tree's [`kernel-prebuilt/Makefile`](https://github.com/Star-caorui/android_device_xiaomi_gold/blob/lineage-24.0/kernel-prebuilt/Makefile) lists them. The DTB and DTBO are the stock ones either way.

## Where the kernel and the proprietary files come from

From Xiaomi's stock package. Which one, how to check the files against it and how to make them yourself is in the READMEs of [android_device_xiaomi_gold-kernel](https://github.com/Star-caorui/android_device_xiaomi_gold-kernel/tree/lineage-24.0#readme) and [android_vendor_xiaomi_gold](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0#readme).
