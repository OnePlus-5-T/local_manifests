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
$ repo init -u https://android.googlesource.com/platform/manifest -b android-13.0.0_r41
$ cd .repo
$ git clone --branch a13/gl https://github.com/roberto-sartori-gl/local_manifests.git local_manifests
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
