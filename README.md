## Device Picture
![Fairphone 4](https://i.imgur.com/B0Sy2T7.jpeg)

## Device Specifications

| Device                   | Fairphone 4                                     |
| -----------------------: | :---------------------------------------------- |
| SoC                      | Qualcomm Snapdragon 750G (SM7225)               |
| QCOM Board Platform      | lito                                            |
| CPU                      | (2) x 2.2 GHz Kryo 570 & (6) x 1.8 GHz Kryo 570 |
| GPU                      | Adreno 619                                      |
| Memory                   | 6GB / 8GB RAM                                   |
| Shipping Android version | 11                                              |
| Storage                  | 128GB / 256GB                                   |
| Battery                  | Removable Li-Ion 3905 mAh                       |
| Dimensions               | 162 x 75.5 x 10.5 mm                            |
| Display                  | 2340 x 1080 (19.5:9), 6.3 inch                  |
| Rear camera 1            | 48MP, f/1.6 (Wide) Dual LED flash               |
| Rear camera 2            | 48MP, f/2.2 (Ultra Wide)                        |
| Rear camera 3            | TOF 3D (Depth)                                  |
| Front camera             | 25MP, f/2.2 (Wide)                              |
| Network                  | GSM / HSPA / LTE / 5G                           |


### Introduction
This is the device tree to build LineageOS 18.1 for the Fairphone 4.


Since the /e/ foundation published the [sources for FP4 support](https://gitlab.e.foundation/e/devices/android_device_fairphone_FP4) this was used as base. The /e/ foundation's [kernel](https://gitlab.e.foundation/e/devices/android_kernel_fairphone_FP4)
Advantages are:
* We can easily sync between the repos.
* Nearly everything is working with it incl. AVB, SELinux etc.

### Current Status
* It builds successfully. :heavy_check_mark:
* Device boots and adb can be accessed. :heavy_check_mark:
* Bootanimation is shown. :heavy_check_mark:
* LineageOS is booting completely. :heavy_check_mark:
* Working things after quick test:
  * Display / Touchscreen :heavy_check_mark:
  * Sound :heavy_check_mark:
  * Bluetooth :heavy_check_mark:
  * Camera :heavy_check_mark:
  * Wi-Fi :heavy_check_mark:
  * NFC :heavy_check_mark:
  * Device encryption :heavy_check_mark:
  * Fingerprint sensor :heavy_check_mark:
  * LTE :heavy_check_mark:
  * NR  ?
  * GPS ?

### Known Issues


### Kernel Source
Kernel is taken from /e/ project.

It has added the qcom specific audio-kernel stuff in techpack/audio and the prima WLAN
drivers.

Kernel sources are here in the lineage-18.1 branch: 
<https://github.com/WeAreFairphone/android_kernel_fairphone_sm7225>


### How to build
* Follow the first steps for setting up the LineageOS build system as described e.g. [here](https://wiki.lineageos.org/devices/FP3/build). Be aware that a complete build can occupy up to 200G of disk space!
* Before downloading the source code using repo sync, create a local manifest file in `.repo/local_manifests`. From the
top of the source tree this command can be used:
```sh
mkdir -p .repo/local_manifests
cat <<EOF > .repo/local_manifests/roomservice.xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
  <project name="WeAreFairphone/android_device_fairphone_FP4" path="device/fairphone/FP4" revision="lineage-18.1" remote="github" />
  <project name="WeAreFairphone/android_kernel_fairphone_sm7225" path="kernel/fairphone/sm7225" revision="lineage-18.1" remote="github" />
  <project name="LineageOS/android_external_bson" path="external/bson" remote="github" />
  <project name="LineageOS/android_system_qcom" path="system/qcom" remote="github" />
  <remove-project name="LineageOS/android_external_chromium-webview"/>
  <project path="external/chromium-webview" name="LineageOS/android_external_chromium-webview" groups="pdk" revision="master" />
  <project name="lineageos4microg/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="master" />
</manifest>
EOF
```
* Do `repo sync -c` to download all needed project repositories.
* Then do
```sh
. build/envsetup.sh
brunch FP4 eng
```

### Use docker to build
Alternatively the [docker image](https://github.com/lineageos4microg/docker-lineage-cicd) from the microG project can be used for building more easily.
It supports building with and without the microG patches.

Go to a preferred work directory and create the required folders if not already done:
```sh
mkdir cache keys lineage logs manifests zips
```

Create the local manifest file in manifest directory:
```sh
cat <<EOF > manifests/roomservice.xml
<?xml version="1.0" encoding="UTF-8"?><manifest>
  <project name="WeAreFairphone/android_device_fairphone_FP4" path="device/fairphone/FP4" revision="lineage-18.1" remote="github" />
  <project name="WeAreFairphone/android_kernel_fairphone_sm7225" path="kernel/fairphone/sm7225" revision="lineage-18.1" remote="github" />
  <project name="LineageOS/android_external_bson" path="external/bson" remote="github" />
  <project name="LineageOS/android_system_qcom" path="system/qcom" remote="github" />
  <remove-project name="LineageOS/android_external_chromium-webview"/>
  <project path="external/chromium-webview" name="LineageOS/android_external_chromium-webview" groups="pdk" revision="master" />
  <project name="lineageos4microg/android_prebuilts_prebuiltapks" path="prebuilts/prebuiltapks" remote="github" revision="master" />
</manifest>
EOF
```

Then only the docker command with correspondig parameters needs to be executed:
```sh
docker run \
    -e "BRANCH_NAME=lineage-18.1" \
    -e "DEVICE_LIST=FP4" \
    -e "SIGN_BUILDS=false" \
    -e "INCLUDE_PROPRIETARY=false" \
    -e "CLEAN_AFTER_BUILD=false" \
    -v "lineage:/srv/src" \
    -v "$(pwd)/zips:/srv/zips" \
    -v "$(pwd)/logs:/srv/logs" \
    -v "$(pwd)/cache:/srv/ccache" \
    -v "$(pwd)/keys:/srv/keys" \
    -v "$(pwd)/manifests:/srv/local_manifests" \
    lineageos4microg/docker-lineage-cicd
```
This automatically downloads the docker image the first time. Afterwards the docker image is started and the build process begins.
This may take some time, especially on first build. Detailed logs are written to the `logs` folder. This can be followed, e.g. with `tail -f logs/...`

If used more regularly it makes sense to put the command into a shell script.
This also allows easier modification of the parameters.

To build with microG patches and pre-installed F-Droid etc., add these parameter lines to the command:
```sh
    -e "SIGNATURE_SPOOFING=restricted" \
    -e "CUSTOM_PACKAGES=GmsCore GsfProxy FakeStore MozillaNlpBackend NominatimNlpBackend com.google.android.maps.jar FDroid FDroidPrivilegedExtension " \
```

More settings and descriptions can be found in the [README](https://github.com/lineageos4microg/docker-lineage-cicd/blob/master/README.md) of the docker image.

### How to install

#### With TWRP
The generated update package can be flashed with TWRP. TWRP flashes it to the
currently inactive slots and activates it afterwards.
The built package can be found in `out/target/product/FP4` with a name similar to `lineage-16.0-20220207-UNOFFICIAL-FP4.zip`.
Alternatively the package can also be taken from an UNOFFICIAL release.

Boot TWRP from bootloader:
```sh
fastboot boot twrp_image.img
```

In TWRP you can sideload the package then. Go to Advanced -> ADB Sideload ->
Swipe
Run
```sh
adb sideload lineage-16.0-20220207-UNOFFICIAL-FP4.zip
```
Adapt the file name of course. This should flash it to the inactive slot and
activate it on success.

Alternatively you can push the package to sd-card and install it from TWRP:
```sh
adb push lineage-16.0-20220207-UNOFFICIAL-FP4.zip /sdcard
```

Optional when coming from stock firmware: Format data from TWRP.

If this is not done LineageOS will reboot and ask you for it.

Reboot and LineageOS should boot up.

#### With fastboot

With the move of fastboot to userspace many partitions need to be loaded from fastbootd instead of fastboot. The `boot.img` file still needs to be loaded from the hardware bootloader

```sh
fastboot flash boot out/target/product/FP4/boot.img
```

Then reboot into fastbootd with
```sh
fastboot reboot fastboot
```

The image files can also be flashed directly with fastbootd.
```sh
fastboot flash recovery out/target/product/FP4/recovery.img
fastboot flash system out/target/product/FP4/system.img
fastboot flash vendor out/target/product/FP4/vendor.img
fastboot flash product out/target/product/FP4/product.img
fastboot flash boot out/target/product/FP4/boot.img
fastboot flash vbmeta out/target/product/FP4/vbmeta.img
```
SPW: FIXME are there other images that need to be loaded directly?

If coming from stock firmware formating data is recommended to prevent LineageOS
from asking for it:
```sh
fastboot -w
```

### How to update
So far OTA update are not available and are untested as well.
That means updates need to be done manually. For that boot to recovery mode
and simply sideload the new package.

Alternatively TWRP can be booted and the new package can be installed from
there. Of course flashing images directly with fastboot is also possible.

### How to extract proprietary blobs
At the moment we manage blobs in a dedicated repository. If you wish to use a different set of
blobs, then we can offer you the following hints.

As an example stock 110 release [firmware dump](https://androidfilehost.com/?fid=4349826312261714249) from k4y0z can be used. SPW FIXME add new example

The FP4 uses a super partition, which needs to be unpacked. Do this with lpunpack, and then mount the required images:
```sh
lpunpack super
sudo mount -t ext4 -o loop,ro system_a.img system
sudo mount -t ext4 -o loop,ro system_ext_a.img system/system_ext
sudo mount -t ext4 -o loop,ro vendor_a.img system/vendor/
sudo mount -t ext4 -o loop,ro product_a.img system/product/
```

