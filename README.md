# Project

This projects aims to create an upgradable AOSP build system for the Oneplus 5 (Cheeseburger).
No repository from AOSP is changed: no commit over AOSP is at the moment present on these sources.

All the changes are provided in separate repositories: this allows to upgrade Android with minimal effort, potentially also for future major versions.

Most of the added repositories is provided by LineageOS or CAF, so thanks to them for those repos.

The ROM supports Lazy kernel (see 'gl/lazy/custom-develop' branch on the kernel repo). Original Lazy Kernel sources:
https://github.com/ederekun/lazy_kernel_op5-t/tree/custom-develop

# Build instructions
Follow the instructions from Google to setup a machine to build Android 11:
https://source.android.com/setup/build/initializing

Then, sync all the sources:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r34
$ cd .repo
$ git clone https://github.com/robertoglxda/local_manifests.git local_manifests
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
$ make otapackage -j12
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

