Westeros WPE Image for HiKey board
==================================

wpe-morty branch of lhg-oe-manifests repository facilitates creating an image for HiKey board with Westeros Compositor, Westeros sample applications, and Metrological Webkit for Wayland browser. The userland is 32-bit, while the kernel is 64-bit. The image is based on Yocto Project Morty release.

The documentation on Morty release can be found at:
https://www.yoctoproject.org/documentation

Along with the manifest, the lhg-oe-manifests repository also contains setup scripts for the OE RPB buildsystem.
The OE RPB buildsystem is using various components from the Yocto Project, most importantly the Openembedded buildsystem, the bitbake task executor and various application and BSP layers.

Initial Configuration
---------------------

To configure the scripts and download the build metadata, do:
```
$ mkdir ~/bin
$ PATH=~/bin:$PATH

$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory. To check out the current branch, specify it with -b:
```
$ repo init -u https://github.com/linaro-home/lhg-oe-manifests.git -b wpe/morty
```
When prompted, configure Repo with your real name and email address.

A successful initialization will end with a message stating that Repo is initialized in your working directory. Your client directory should now contain a .repo directory where files such as the manifest will be kept.

To pull down the metadata sources to your working directory from the repositories as specified in the default manifest, run
```
$ repo sync
```
When downloading from behind a proxy (which is common in some corporate environments), it might be necessary to explicitly specify the proxy that is then used by repo:
```
$ export HTTP_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
$ export HTTPS_PROXY=http://<proxy_user_id>:<proxy_password>@<proxy_server>:<proxy_port>
```
More rarely, Linux clients experience connectivity issues, getting stuck in the middle of downloads (typically during "Receiving objects"). It has been reported that tweaking the settings of the TCP/IP stack and using non-parallel commands can improve the situation. You need root access to modify the TCP setting:
```
$ sudo sysctl -w net.ipv4.tcp_window_scaling=0
$ repo sync -j1
```
Setup Environment
-----------------

Run
```
$ . setup-environment
```
and select `hikey-32` as the MACHINE, and `rpb-wayland` as DISTRO.

Another option is to specify MACHINE and DISTRO in the command line:
```
$ MACHINE=hikey-32 DISTRO=rpb-wayland source setup-environment
```
Run bitbake to Build the Images
-------------------------------
```
$ bitbake_secondary_image --extra-machine hikey lhg-westeros-wpe-image
```
`bitbake_secondary_image` actually runs two builds. So in the build directory,
under `tmp-*/deploy/images/` two directories are created: one for 32-bit build
artifacts:
```
tmp-rpb_wayland-glibc/deploy/images/hikey-32
```
and the other for the 64-bit ones:
```
tmp-rpb_wayland-glibc/deploy/images/hikey
```
The 64-bit build creates rpb-minimal-image, as we really need only kernel and the modules from there.

Copy 64-bit kernel and the modules to the 32-bit rootfs
-------------------------------------------------------

Copy the 64-bit kernel and the modules (/boot and /lib/modules) into 32-bit rootfs before flashing the image to hikey board as below. We will use images32 and images64 directories to hold the temporary files and the final 32-bit rootfs image to flash into the board.
```
$ mkdir images32
$ mkdir images64
$ cp tmp-rpb_wayland-glibc/deploy/images/hikey/rpb-minimal-image-hikey.ext4.gz images64
$ cp tmp-rpb_wayland-glibc/deploy/images/hikey-32/lhg-westeros-wpe-image-hikey-32.ext4.gz images32
$ gunzip -k images32/lhg-westeros-wpe-image-hikey-32.ext4.gz
$ gunzip -k images64/rpb-minimal-image-hikey.ext4.gz
```
Resize the 32-bit rootfs image to get some more free space for the 64-bit kernel and the modules:
```
$ resize2fs images32/lhg-westeros-wpe-image-hikey-32.ext4 768M
```
Mount the 64-bit image under rootfs64 directory:
```
$ mkdir rootfs64
$ sudo mount -o loop,sync,rw images64/rpb-minimal-image-hikey.ext4 rootfs64
```
Mount the 32-bit image under rootfs-32 directory:
```
$ mkdir rootfs32
$ sudo mount -o loop,sync,rw images32/lhg-westeros-wpe-image-hikey-32.ext4 rootfs32
```
Copy the required files with rsync:
```
$ sudo rsync -avz rootfs64/boot rootfs32/
$ sudo rsync -avz rootfs64/lib/modules rootfs32/lib/
```
(Optional) Check that the files have been copied to the rootfs32:
```
$ ls rootfs32/boot
$ ls rootfs32/lib/modules/
```
Unmount the 32-bit and the 64-bit images:
```
$ sudo umount rootfs64
$ sudo umount rootfs32
```
Now the lhg-westeros-wpe-image-hikey-32.ext4 image has the 64-bit kernel and the modules in it.
Create the sparse image for flashing with fastboot:
```
$ ext2simg images32/lhg-westeros-wpe-image-hikey-32.ext4 images32/lhg-westeros-wpe-image-hikey-32.img
```

Flashing Instructions for HiKey
-------------------------------
Follow [this link](https://github.com/96boards/documentation/wiki/HiKeyUEFI#flash-binaries-to-emmc-) for standard flashing procedure.

If only the rootfs has changed since the previous build, and all the images from the previous build were flashed into the board, then the following command is enough (J15 Link 5-6 should be closed):
```
$ sudo fastboot flash system images32/lhg-westeros-wpe-image-hikey-32.img
```

Running Westeros and WPE
------------------------

Boot HiKey with HDMI out connected to the monitor, and keyboard, mouse (use USB hub) and USB-to-Ethernet adaptor connected to the USB ports. Connect the debug UART to the host and run these commands from the serial console to launch Westeros compositor:
```
$ export XDG_RUNTIME_DIR=/run/user/
$ LD_PRELOAD=/usr/lib/libwesteros_gl.so.0 westeros --renderer /usr/lib/libwesteros_render_gl.so.0 --enableCursor --display westeros-1-0 &
```
After that you should see a blank screen with mouse cursor on it. The option `--enableCursor` option is to enable the mouse pointer, and the `--display <displayName>` is used to set the display name.

To run Westeros test application execute:
```
$ westeros_test --display=westeros-1-0
```
You should see rotating color triangle on the screen.

To run the WPELauncher with westeros backend do:
```
$ export WAYLAND_DISPLAY=westeros-1-0
$ export WPE_BACKEND=westeros
$ ./WPELauncher <URL>
```
This will launch the WPE browser with the web page from the provided URL. If no `<URL>` is specified at the command line the browser opens youtube.com. Make sure that HiKey has internet connection before running `WPELauncher`.
