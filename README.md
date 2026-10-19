# local_manifests

The local manifest for building LineageOS 23.2 (Android 16) for the Redmi Note 13 5G (`gold`), and how to build it.

Releases, and what works on them, are in [lineageos-gold](https://github.com/Star-caorui/lineageos-gold).

## What gold.xml adds

| Path | Repository |
| --- | --- |
| `device/xiaomi/gold` | the [device tree](https://github.com/Star-caorui/android_device_xiaomi_gold/tree/lineage-23.2) |
| `device/xiaomi/gold-kernel` | the [stock kernel and its modules](https://github.com/Star-caorui/android_device_xiaomi_gold-kernels/tree/lineage-23.2), prebuilt |
| `vendor/xiaomi/gold` | the [proprietary files](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-23.2) |
| `device/mediatek/sepolicy_vndr`, `hardware/mediatek`, `hardware/xiaomi` | LineageOS |

## Building

You need `repo` and `git-lfs`, plus the usual [LineageOS build dependencies](https://wiki.lineageos.org/devices/).

```sh
repo init -u https://github.com/LineageOS/android.git -b lineage-23.2 --git-lfs
git clone -b lineage-23.2 https://github.com/Star-caorui/local_manifests .repo/local_manifests
repo sync -c

source build/envsetup.sh
breakfast gold userdebug
m bacon
```

The output is in `out/target/product/gold/`.

## Where the kernel and the proprietary files come from

From Xiaomi's stock package. Which one, how to check the files against it and how to make them yourself is in the READMEs of [android_device_xiaomi_gold-kernels](https://github.com/Star-caorui/android_device_xiaomi_gold-kernels/tree/lineage-23.2#readme) and [android_vendor_xiaomi_gold](https://github.com/Star-caorui/android_vendor_xiaomi_gold/tree/lineage-23.2#readme).

## Build hosts with little RAM

`host/build.sh` in [lineageos-gold](https://github.com/Star-caorui/lineageos-gold/tree/main/host) is what we use on an 8-core, 16 GB machine. Run it from the top of the Android tree, or set `SOURCE_TREE`. It does the same as the steps above, turns on ccache if it is installed, and applies one local patch. A host with enough memory does not need it.

- `soong-memory-env.patch` (`build/soong`): passes `GOGC` and `GOMEMLIMIT` through to soong_build. The script sets them to 50 and 10GiB unless the environment says otherwise, which keeps soong_build at 20 to 23 GB.

Restore the project before `repo sync`: `git -C build/soong checkout .`

The part of soong_build that does not fit in RAM is swapped, and where it goes decides how long Soong's analysis takes. With the defaults of Ubuntu 24.04 (kernel 6.8) the analysis took 2 h 45 min to 3 h 25 min on that machine. With the swapped pages kept compressed in memory it took 15 minutes:

```
echo zsmalloc > /sys/module/zswap/parameters/zpool
echo lz4      > /sys/module/zswap/parameters/compressor
echo 80       > /sys/module/zswap/parameters/max_pool_percent
echo N        > /sys/module/zswap/parameters/shrinker_enabled   # no writeback to the swap file
echo Y        > /sys/module/zswap/parameters/enabled
echo Y        > /sys/module/zswap/parameters/exclusive_loads   # kernels before 6.9; later ones always do this
echo 0        > /proc/sys/vm/page-cluster
```

`exclusive_loads` was added after that measurement. It drops a page from the compressed pool as soon as it is read back, instead of keeping it in both places. On LineageOS 24.0's analysis, with 22 GB of RAM, it took 7% off (11 min 27 s to 10 min 39 s); other compressors, a larger `page-cluster` and zram were all slower.

zswap still needs a swap file behind it; ours was 16 GB, and the analysis used 13 GB of it without writing anything to disk. We run the build in a systemd unit with `MemoryMax=14000M`, `MemorySwapMax=15G` and `OOMPolicy=stop`, so that a build that outgrows the machine is killed instead of taking the machine down.
