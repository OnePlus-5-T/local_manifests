# Project

This projects aims to create an upgradable AOSP build system for the OnePlus 5 (Cheeseburger) and the OnePlus 5T (Dumpling).
No repository from AOSP is changed: no commit over AOSP is at the moment present on these sources.

All the changes are provided in separate repositories: this allows to upgrade Android with minimal effort, potentially also for future major versions.

Most of the added repositories is provided by LineageOS or CAF, so thanks to them for those repos.

# Build instructions
Follow the instructions from Google to setup a machine to build Android 13:
https://source.android.com/setup/build/initializing

Then, sync all the sources:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-13.0.0_r52
$ cd .repo
$ git clone --branch 4.14/a13/gl https://github.com/roberto-sartori-gl/local_manifests.git local_manifests
$ cd ..
$ repo sync -c --no-clone-bundle --no-tags
```
then:
```
$ source build/envsetup.sh
$ lunch aosp_cheeseburger-user
$ make -j12
```
the images will be available in `out/target/product/cheeseburger`.

To build an OTA:
```
$ source build/envsetup.sh
$ lunch aosp_cheeseburger-user
$ make -j12
$ make otatools-package -j12
$ make otapackage_custom -j12
```
and the OTA zip will be available in `out/target/product/cheeseburger`.

# Flash instructions
The images can be flashed using fastboot:
```
$ fastboot flash boot out/target/product/cheeseburger/boot.img
$ fastboot flash recovery out/target/product/cheeseburger/recovery.img
$ fastboot flash system out/target/product/cheeseburger/system.img
$ fastboot flash vendor out/target/product/cheeseburger/vendor.img
```
Format data if this is a first flash:
```
$ fastboot format userdata
```
Note that not all the fastboot versions support the 'format' command.

The OTA can be flashed from the recovery (using the 'ADB sideload' feature) or using third party recoveries like TWRP.

To build for Dumpling, run the same commands with 'dumpling' instead of 'cheeseburger' in commands and paths.

# Build the kernel
The ROM sources include a prebuilt kernel. To build a kernel from sources and update it in the Android sources, follow these instructions.

Download the sources and the cross compilers (needed only the first time):
```
$ cd ~
$ mkdir kernel_build
$ cd kernel_build
$ git clone --single-branch --branch 4.14.284/msm8998_op_new https://github.com/roberto-sartori-gl/4.14-kernel-oneplus-msm8998 kernel_oneplus_msm8998
$ git clone --depth 1 -b android12L-release --single-branch https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/ prebuilt_aarch64
$ git clone --depth 1 -b android12L-release --single-branch https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/ prebuilt_arm
$ git clone --depth 1 -b android-13.0.0_r0.70 --single-branch https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86 prebuilt_clang
$ git clone --depth 1 -b android-13.0.0_r0.70 --single-branch https://android.googlesource.com/platform/prebuilts/gas/linux-x86 prebuilt_clang_triple
$ git clone --depth 1 -b android-13.0.0_r6 --single-branch https://android.googlesource.com/platform/prebuilts/build-tools prebuilt_tools
```

Build the kernel:
```
$ CC_ARM32_PATH=~/kernel_build/prebuilt_arm/bin
$ CC_ARCH64_PATH=~/kernel_build/prebuilt_aarch64/bin
$ CLANG_PATH=~/kernel_build/prebuilt_clang/clang-r458507/bin
$ CLANG_BIN=${CLANG_PATH}/clang
$ CLANG_TRIPLE_BIN=~/kernel_build/prebuilt_clang_triple/aarch64-linux-gnu-
$ MAKE_PATH=~/kernel_build/prebuilt_tools/linux-x86/bin
$ PATH="${PATH}:${CLANG_PATH}"
$ DEFCONFIG=msm8998_oneplus_android_defconfig

$ cd ~/kernel_build/kernel_oneplus_msm8998
$ git pull
$ ${MAKE_PATH}/make O=out ARCH=arm64 CC=${CLANG_BIN} LLVM=1 LLVM_IAS=1 ${DEFCONFIG} -j8
$ ${MAKE_PATH}/make O=out ARCH=arm64 CROSS_COMPILE_ARM32=${CC_ARM32_PATH}/arm-linux-androidkernel- CROSS_COMPILE=${CC_ARCH64_PATH}/aarch64-linux-androidkernel- CC=${CLANG_BIN} LLVM=1 LLVM_IAS=1 CLANG_TRIPLE=${CLANG_TRIPLE_BIN} -j6
```

# Extra patches
Some patches are needed over AOSP to fix specific issues.
At the moment, 2 patches are needed:
1) To avoid crashes with some Gapps versions:
https://github.com/crdroidandroid/android_external_setupcompat/commit/0450c92cbe5c75cc31d4cdc0958918836b28ca53
2) To fix issues with SIM 2:
https://github.com/crdroidandroid/android_frameworks_opt_telephony/commit/cbded5de818d1d23ea8ac7a67f178a95da64a9f0
3) To enable the styles and colors selection under Settings -> Wallpaper & style:
https://github.com/Flamingo-OS/packages_apps_Launcher3/commit/e6d6b64264ef45a72791347d9f39d85ff412e58f
https://github.com/LineageOS/android_packages_apps_ThemePicker/commit/14584d051eb8959e8bfb3b29173a0c40b9e991c5
4) To fix a Dialer crash when opening Settings -> Display options:
https://github.com/RiceDroid/android_packages_apps_Dialer/commit/8c4ab1faf4d534d32f570c4bf9cf002ab3ad20d0
5) Fix offline charging 'always on display' issue:
https://android-review.googlesource.com/c/platform/bootable/recovery/+/2205736/

These patches are not necessary to boot or use Android.
