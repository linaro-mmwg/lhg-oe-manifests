# Manifests for public LHG OE builds based on jethro

# Build instructions
Make sure that recent repo is used:
```
$ test -d $HOME/bin || mkdir -p $HOME/bin
$ wget -q --no-check-certificate "http://android.git.linaro.org/gitweb?p=tools/repo.git;a=blob_plain;f=repo;hb=refs/heads/stable" -O $HOME/bin/repo
$ chmod a+x $HOME/bin/repo
$ export PATH=$HOME/bin:$PATH
```
The last command might not be needed (if $HOME/bin is already in the $PATH).

Download the OE data:
```
$ repo init -u https://github.com/linaro-home/lhg-oe-manifests.git -b jethro -m ${board_type}.xml
$ repo sync
```
.. where ${board_type} is one of the following:
* dragonboard-410c
  (DragonBoard 410c by Qualcomm)
* am57xx-evm
  (AM57xx EVM, BeagleBoard-X15 by TI)

Prepare the build environment (the script will ask to select the machine
from the list):
```
$ source meta-lhg/script/envsetup.sh
```
.. or instead, the machine to build for can be specified in the command line:
```
$ MACHINE=dragonboard-410c source meta-lhg/script/envsetup.sh
```
The setup script creates the directory named "build" in the current one, and
cd's into it.

Then build the image:
```
$ bitbake ${image_type}
```
.. where ${image_type} is one of the following:
* lof-weston-image (basic Wayland image with Weston)
* lof-mm-image (same as above plus Gstreamer)
* lof-chromium-image (same as above plus Chromium)

The images are created in tmp-glibc/deploy/images/${board_type}/ directory.

# Flasing the images into DragonBoard 410c
This guide assumes that the Linux Bootloaders and eMMC partition layout are
used like described here:
https://github.com/96boards/documentation/wiki/Dragonboard-410c-OpenEmbedded-and-Yocto#bootloaders-and-emmc-partitions

Inside the tmp-glibc/deploy/images/dragonboard-410c/ directory find the boot and the rootfs images:
* boot-dragonboard-410c.img
* ${image_type}-dragonboard-410c.ext4.gz (e.g. lof-chromium-image-dragonboard-410c.ext4.gz)

Unzip the rootfs image and convert it into fastboot sparse image:
```
$ gunzip ${image_type}-dragonboard-410c.ext4.gz
$ ext2simg ${image_type}-dragonboard-410c.ext4 ${image_type}-dragonboard-410c.img
```
Flash the images using fastboot (use the micro USB connector on the db410c board, and power on the board while keeping the S4 "(-)" button pressed to get into fastboot mode):
```
$ fastboot flash boot boot-dragonboard-410c.img
$ fastboot flash rootfs ${image_type}-dragonboard-410c.img
```
After flashing is complete, powercycle the board to boot into weston.

# Flashing the images into BeagleBoard-X15
[TBD]
