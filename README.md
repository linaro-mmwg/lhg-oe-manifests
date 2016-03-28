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
$ repo init -u https://github.com/linaro-home/lhg-oe-manifests.git -b jethro -m default.xml
$ repo sync
```
Before starting the build the userland Mali drivers for HiKey board must be
downloaded from here:

https://drive.google.com/a/linaro.org/file/d/0B8Uq4Q7WAxO4ZjJLdGJQR01DRkE/view?usp=sharing

... and the tarball must be placed in the following folder:
```
~/Public/oe-downloads/
```
Prepare the build environment (the script will ask to select the machine
from the list):
```
$ source meta-lhg/script/envsetup.sh
```
The machine list is created automatically using the contents of the BSP
layers, and is quite long. The currently supported machines are:
* dragonboard-410c
  (DragonBoard 410c by Qualcomm)
* hikey
  (HiKey board from HiSilicon)

Another option is to specify the machine to build for in the command line:
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

The images are created in tmp-glibc/deploy/images/${MACHINE}/ directory.

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

# Flashing the images into HiKey
For HiKey we use a customised grub configuration.

Get the boot-fat.uefi.img.gz from here:

https://builds.96boards.org/snapshots/hikey/linaro/debian/latest/

Mount the image at boot-fat directory:
```
$ mkdir -p boot-fat
$ sudo mount -o loop,rw,sync boot-fat.uefi.img boot-fat
```
Locate the gru.cfg file inside boot-fat/EFI/BOOT/ directory, and change the
'Debian GNU/Linux (eMMC)' section to look like:
```
menuentry 'Debian GNU/Linux (eMMC)' {
    search.fs_label system root
    set root=($root)
    linux /boot/Image video=1920x1080@25 console=tty0 console=ttyAMA3,115200 root=/dev/mmcblk0p9 rootwait rw earlyprintk efi=noruntime
    devicetree /boot/hi6220-hikey.dtb
}
```
The grub.cfg is configured to 1920x1080@25 resolution. If that doesn't works
for you for some reason (may depend on the monitor used), try other modes.
800x600@60 usually works, and can be used as a starting point.

Then unmount the image, and flash the modified image into the board (use the
micro USB connector on the HiKey board, and place a jumper over pins 5-6 on J15
to put the board into fastboot mode):
```
$ sudo umount boot-fat.uefi.img
$ fastboot flash boot boot-fat.uefi.img
```
Inside the tmp-glibc/deploy/images/hikey/ directory find the rootfs image:
* ${image_type}-hikey.ext4.gz (e.g. lof-chromium-image-hikey.ext4.gz)

Unzip the rootfs image and convert it into fastboot sparse image:
```
$ gunzip ${image_type}-hikey.ext4.gz
$ ext2simg ${image_type}-hikey.ext4 ${image_type}-hikey.img
```
Flash the images using fastboot (use the micro USB connector on the HiKey board,
 and place a jumper over pins 5-6 on J15 to put the board into fastboot mode):
```
$ fastboot flash system ${image_type}-hikey.img
```
After the image is written, remove the jumper from pins 5-6, and reboot.
Disconnect the cable from the micro USB connector used for flashing the board,
make sure that mouse and keyboard are connected to the board (weston would not
start if no input device is connected).

# Flashing the images into BeagleBoard-X15
[TBD]
