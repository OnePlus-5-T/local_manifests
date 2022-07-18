# Project

This projects aims to create an upgradable AOSP build system for the OnePlus 5 (Cheeseburger) and the OnePlus 5T (Dumpling).
No repository from AOSP is changed: no commit over AOSP is at the moment present on these sources.

All the changes are provided in separate repositories: this allows to upgrade Android with minimal effort, potentially also for future major versions.

Most of the added repositories is provided by LineageOS or CAF, so thanks to them for those repos.

# Build instructions
Follow the instructions from Google to setup a machine to build Android 12:
https://source.android.com/setup/build/initializing

Then, sync all the sources:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-12.1.0_r10
$ cd .repo
$ git clone --branch a12/gl https://github.com/roberto-sartori-gl/local_manifests.git local_manifests
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
$ make otapackage_norecovery -j12
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

To build for Dumpling, run the same commands with 'dumpling' instead of 'cheeseburger' in commands an paths.

# Build the kernel
The ROM sources include a prebuilt kernel. To build a kernel from sources and update it in the Android sources, follow these instructions.

Download the sources and the cross compilers (needed only the first time):
```
$ cd ~
$ mkdir kernel_build
$ cd kernel_build
$ git clone https://github.com/roberto-sartori-gl/kernel_oneplus_msm8998.git kernel_oneplus_msm8998
$ git clone --depth 1 -b android12L-release --single-branch https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/aarch64/aarch64-linux-android-4.9/ prebuilt_aarch64
$ git clone --depth 1 -b android12L-release --single-branch https://android.googlesource.com/platform/prebuilts/gcc/linux-x86/arm/arm-linux-androideabi-4.9/ prebuilt_arm
$ git clone --depth 1 -b android-12.1.0_r4 --single-branch https://android.googlesource.com/platform/prebuilts/clang/host/linux-x86 prebuilt_clang
$ git clone --depth 1 -b android-12.1.0_r4 --single-branch  https://android.googlesource.com/platform/prebuilts/build-tools prebuilt_tools

$ cd ~/kernel_build/kernel_oneplus_msm8998
$ git checkout gl/lineage-19.1
```

Build the kernel:
```
$ cd ~/kernel_build/kernel_oneplus_msm8998
$ git pull
$ CC_ARM32_PATH=~/kernel_build/prebuit_arm/bin
$ CC_ARCH64_PATH=~/kernel_build/prebuilt_aarch64/bin
$ CC_CLANG_PATH=~/kernel_build/prebuilt_clang/clang-r416183b1/bin
$ MAKE_PATH=~/kernel_build/prebuilt_tools/linux-x86/bin

$ ${MAKE_PATH}/make -j8 ARCH=arm64 CROSS_COMPILE_ARM32=arm-linux-androidkernel- CROSS_COMPILE=aarch64-linux-android- CC=clang CLANG_TRIPLE=aarch64-linux- PATH="${CC_ARM32_PATH}:${CC_ARCH64_PATH}:${CC_CLANG_PATH}:${PATH}" O=out lineage_oneplus5_defconfig
$ ${MAKE_PATH}/make -j8 ARCH=arm64 CROSS_COMPILE_ARM32=arm-linux-androidkernel- CROSS_COMPILE=aarch64-linux-android- CC=clang CLANG_TRIPLE=aarch64-linux- PATH="${CC_ARM32_PATH}:${CC_ARCH64_PATH}:${CC_CLANG_PATH}:${PATH}" O=out
```
Copy the Image.gz-dtb file from the prebuilt directory to the Android build system:
```
$ cp ~/kernel_build/kernel_oneplus_msm8998/out/arch/arm64/boot/Image.gz-dtb ~/aosp_build_system/kernel/oneplus/prebuilt/Image.gz-dtb
```

Build Android again to update the kernel.

# Extra patches
Some patches are needed over AOSP to fix specific issues.
At the moment, 2 patches are needed:
1) To avoid crashes with some Gapps versions: https://github.com/crdroidandroid/android_external_setupcompat/commit/0450c92cbe5c75cc31d4cdc0958918836b28ca53
2) To fix issues with SIM 2: https://github.com/crdroidandroid/android_frameworks_opt_telephony/commit/cbded5de818d1d23ea8ac7a67f178a95da64a9f0

These patches are not necessary to boot or use Android.
