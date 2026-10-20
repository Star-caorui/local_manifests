# local_manifests

The local manifest for building LineageOS 24.0 (Android 17) for the Redmi Note 13 5G (`gold`), and how to build it.

Releases, and what works on them, are in [lineageos-gold](https://github.com/Star-caorui/lineageos-gold).

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-24.0) |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0) |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |
| `lineage-sdk`, `packages/apps/Updater`, `packages/modules/Bluetooth` | LineageOS's, with one change each that is under review on LineageOS's Gerrit ([508379](https://review.lineageos.org/c/LineageOS/android_lineage-sdk/+/508379), [508380](https://review.lineageos.org/c/LineageOS/android_packages_apps_Updater/+/508380), [508408](https://review.lineageos.org/c/LineageOS/android_packages_modules_Bluetooth/+/508408)), from our forks until it is merged |

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/LineageOS/android.git -b lineage-24.0 --git-lfs
git clone -b lineage-24.0 https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c

mkdir -p out-kernel/xiaomi_gold/.repo && touch out-kernel/xiaomi_gold/.out-dir
cp -R .repo/repo out-kernel/xiaomi_gold/.repo/repo
(cd out-kernel/xiaomi_gold &&
  repo init -u https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest -b lineage-24.0 --depth=1 --no-use-superproject &&
  repo sync -c --no-tags)

export SKIP_KERNEL_SYNC=1
source build/envsetup.sh
breakfast gold userdebug
m bacon
```

The output is in `out/target/product/gold/`.

The kernel is built as for LineageOS's lamu: `breakfast` runs `build_kernel`, which runs [android_kernel_xiaomi_gold_manifest](https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest/tree/lineage-24.0#readme)'s Kleaf build in `out-kernel/xiaomi_gold` and puts it into `device/xiaomi/gold-kernels`. It syncs only the kernel manifests LineageOS hosts, so this one is synced by hand, as above (in a `.repo` of its own, as `build_kernel` makes it, or `repo init` finds the Android tree's and initialises that), and `SKIP_KERNEL_SYNC=1` keeps it from trying; after a sync there (`repo sync -c --no-tags`), `build_kernel` builds it again. It needs to reach `android.googlesource.com`, `github.com` and `ci.android.com`.

## Choosing the kernel

Releases use the GKI build stock ships, 15511674. `GKI_BUILD` picks another of the same KMI generation from the kernel manifest's `gki.sha256`, for testing; the vendor modules are the same:

```sh
export GKI_BUILD=16498190 SKIP_KERNEL_SYNC=1
source build/envsetup.sh
breakfast gold userdebug
m bacon
```

## Where the kernel and the proprietary files come from

The kernel image and the GKI modules are Google's builds. The vendor modules, the device tree blob and its overlay are built from source; the [`OS3.0.5.0.VNQMIXM`](https://github.com/Star-caorui/android_kernel_xiaomi_gold_manifest/tree/OS3.0.5.0.VNQMIXM) branch of android_kernel_xiaomi_gold_manifest shows they give stock's modules, DTB and DTBO, byte for byte. The proprietary files are from Xiaomi's stock package; which one, how to check the files against it and how to make them yourself is in the README of [android_vendor_xiaomi_gold](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-24.0#readme).
