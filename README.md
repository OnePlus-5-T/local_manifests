# Project

This projects aims to build Android with kernel 4.14 for the OnePlus 5 (Cheeseburger).
No repository from AOSP is changed: no commit over AOSP is at the moment present on these sources.
This starts from Sony's work:
https://github.com/sonyxperiadev/local_manifests

Thanks to Sony that allowed this!

# Build instructions
Follow the instructions from Google to setup a machine to build Android 11:
https://source.android.com/setup/build/initializing

Then, sync all the sources:
```
$ repo init -u https://android.googlesource.com/platform/manifest -b android-11.0.0_r32
$ cd .repo
$ git clone https://github.com/robertoglxda/local_manifests.git local_manifests
$ cd local_manifests
$ git checkout kernel4.14
$ cd ../..
$ repo sync -c --no-clone-bundle --no-tags
```
then:
```
$ source build/envsetup.sh
$ lunch aosp_cheeseburger-userdebug
$ make -j12
```
the images will be available in `out/target/product/cheeseburger`.

To build an OTA:
```
$ source build/envsetup.sh
$ lunch aosp_cheeseburger-userdebug
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

