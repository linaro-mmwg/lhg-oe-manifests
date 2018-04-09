lhg-oe-manifests
================

The lhg-oe-manifests repository contains the setup scripts for the OE RPB buildsystem (taken from OE RPB repo), along
with a LHG specific manifest.

The purpose of the LHG manifest is manifest is to give working LHG reference platform builds, where the integration of
Chromium, OP-TEE and DRM has already been done and validated. This can then be used as a baseline reference implementation
by Linaro Home Group (LHG) members.

The LHG manifest includes additional OE layers such as meta-lhg and may also override other OE layers such as meta-browser
with LHG specific branches where integration has already been done. Other OE layers containing proprietary DRM systems
are also available.

This repository also contains setup scripts for the OE RPB buildsystem. If you want to (re)build packages or images for OE RPB, this is the thing to use.
The OE RPB buildsystem is using various components from the Yocto Project, most importantly the Openembedded buildsystem, the bitbake task executor and various application and BSP layers.

To configure the scripts and download the build metadata, do:
```
$ mkdir ~/bin
$ PATH=~/bin:$PATH

$ curl http://commondatastorage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
$ chmod a+x ~/bin/repo
```
Run repo init to bring down the latest version of Repo with all its most recent bug fixes. You must specify a URL for the manifest, which specifies where the various repositories included in the Android source will be placed within your working directory. To check out the current branch, specify it with -b:
```
$ repo init -u https://github.com/linaro-home/lhg-oe-manifests.git -b morty/playready
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

MACHINE values can be:
* dragonboard-410c
* hikey

DISTRO values can be:
* rpb
* rpb-wayland

```
$ . setup-environment
$ MACHINE=<machine> DISTRO=<distro> bitbake <image>
```

meta-lhg layer defines two new image types, lhg-westeros-wpe-image and rpb-westonchromium-image.

e.g.
```
MACHINE=hikey DISTRO=rpb-wayland bitbake rpb-westonchromium-image
```

Will build an image for HiKey with Chromium, OP-TEE, supporting Playready DRM
already integrated.

The integration is achieved through the meta-lhg-prop layer which adds various bbapends
and extra recipes for the Playready drm support.

Flashing Instructions for HiKey
-------------------------------
Follow [this link](https://github.com/96boards/documentation/wiki/HiKeyUEFI#flash-binaries-to-emmc-) for standard flashing procedure.

If switching between Android & OE builds remember to flash the new partition table.
Also it is strongly advised to always flash fip.bin (which contains OP-TEE OS & ATF)
to ensure the OP-TEE os is the correct version.

Running Chromium with OpenCDM & Playready TA
--------------------------------------------

On the target use the following commands to start the cdmiservice and run Chromium
with OpenCDM plugin enabled.

```
$ su
$ chmod 666 /dev/te*
$ cd /usr/share/playready
$ cdmiservice &
$ /usr/bin/chromium/chrome --no-sandbox --use-gl=egl --ozone-platform=wayland --no-sandbox --composite-to-mailbox --in-process-gpu --enable-low-end-device-mode --start-maximized --user-data-dir=data_dir --blink-platform-log-channels=Media --register-pepper-plugins="/usr/lib64/chromium/libopencdmadapter.so#ClearKey CDM#ClearKey CDM0.1.0.0#0.1.0.0;application/x-ppapi-open-cdm" http://people.linaro.org/~peter.griffin/dash.js/samples/dash-if-reference-player/index.html
```

Using the dash.js client select one of the bottom two clips from "Microsoft Test Content" section.

Notes
-----

1. Ensure cdmiservice is run from /usr/share/playready directory (required so it finds certifcates).
2. Ensure path to libopencdmadapter.so is correct.
3. Ensure TA is present under /lib/optee_armtz/82dbae9c-9ce0-47e0-a1cb4048cfdb84aa.ta
3. For further debugging enable logging in opencdm plugin and cdmiservice.


Doing mixed 32-bit/64-bit builds
--------------------------------
Follow [these instructions](mixed-build.md) to build 32-bit userland with 64-bit
kernel and kernel modules.

Creating a local topic branch
-----------------------------

If you need to create local branches for all repos which then can be done e.g.
```
$ ~/bin/repo start myangstrom --all
```
Where 'myangstrom' is the name of branch you choose

Updating the sandbox
--------------------

If you need to bring changes from upstream then use following commands
```
$ repo sync
```
Rease your local committed changes
```
$ repo rebase
```
If you find any bugs please report them here

https://github.com/linaro-home/lhg-oe-manifests/issues

If you have questions or feedback, please subscribe to

https://lists.linaro.org/mailman/listinfo/openembedded

Maintainers
-------------------------

* Andrey Konovalov <mailto:andrey.konovalov@linaro.org>
