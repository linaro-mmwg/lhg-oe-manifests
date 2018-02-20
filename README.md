# Multimedia Working Group OpenSDK Build (MMWG OE Build)

Multimedia Working Group OpenSDK Build is the MMWG reference build, where the integration of Chromium, OP-TEE and DRM has already been done and validated. This can then be used as a baseline reference implementation by MWMG members.

Multimedia Working Group OpenSDK is based on [Linaro OpenEmbedded Reference Platform Build (OE RPB)](https://github.com/96boards/oe-rpb-manifest)

## Maintainers

* Andrey Konovalov <mailto:andrey.konovalov@linaro.org>

## Support

If you find any bugs please report them here

https://github.com/linaro-home/lhg-oe-manifests/issues

If you have questions or feedback, please subscribe to

https://lists.linaro.org/mailman/listinfo/openembedded

# Using MMWG OE Build

This document provides instructions to get started with Multimedia Working Group OpenSDK. It tries (when possible) to be generic for any board supported. The board diversity should be addressed through dedicated BSP layer then MACHINE choice.

## Introduction

This is not an introduction on OpenEmbedded or Yocto Project. If you are not familiar with OpenEmbedded and the Yocto Project, it is very much recommended to read the appropriate documentation first. For example, you can start with:
* http://openembedded.org/wiki/Main_Page
* http://yoctoproject.org/
* https://www.yoctoproject.org/documentation

In this wiki, we assume that the reader is familiar with basic concepts of OpenEmbedded.

## MMWG OE Layers
In addition to [OE RPB Layers](https://github.com/96boards/oe-rpb-manifest/blob/rocko/README.md#oe-rpb-layers) MMWG OE manifest includes public OE layers such as meta-lhg and may also override other OE layers such as meta-browser with MMWG specific branches where integration has already been done. Other OE layers containing proprietary DRM systems are available separately.


|   Layer                 |   Description                    |
|:-----------------------:|:----------------------|
| meta-lhg | MMWG components, the sample Wayland Weston image recipe included |
| meta-lhg-integration | Third party recipes, and the changes to integrate them into MMWG OE build |
| meta-wpe | lhg-westeros-wpe-image, and wpewebkit integration changes |

## lhg-oe-manifests repository

These are the setup scripts for the OE RPB buildsystem. If you want to (re)build packages or images for OE RPB, this is the thing to use.
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
$ repo init -u https://github.com/linaro-home/lhg-oe-manifests.git -b thud
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

## Host package Dependencies

In order to successfully set up your build environment, you will need to install the following package dependencies.

**Step 1**: You will need git installed on your Linux host machine

`$ sudo apt-get install git`

**Step 2**: Visit the OpenEmbedded (Getting Started) wiki to see which distribution specific dependencies you will need

http://www.openembedded.org/wiki/Getting_started

Setting up the build environment will first search for `whiptail`, if it is not present then it will search for `dialog`. You only need one of the following packages to ensure your setup-environement runs correctly:

`$ sudo apt-get install whiptail`

or

`$ sudo apt-get install dialog`

**Please Note**: If you are running Ubuntu 16.04 you will need to add the following line to your `/etc/apt/sources.list`

`deb http://archive.ubuntu.com/ubuntu/ xenial universe`

```shell
$ cd /etc/apt/
#vim text editor is used in this example
#sudo is used to allow editing, sources.list is set to read only
$ sudo vim sources.list
```

All required dependencies should now be installed on your host environment, you are ready to begin your build environment setup.

## Setup Environment

It is recommended to use the `setup-environment` script. Among several things, it will prompt the user for the list of supported machines and distros. The setup script will also create (if they don't exsist) the local build configuration files.

To run the setup script:
```
$ . setup-environment
```

And follow the instructions. If you already know the value for MACHINE and DISTRO, it is possible to set them as environment variables, e.g. 
```
$ MACHINE=dragonboard-410c DISTRO=rpb-wayland source setup-environment
```

## Build a sample Wayland/Weston image

For Wayland/Weston, it is needed to select `rpb-wayland` for the DISTRO when running setup-environment. Then you can run a sample image with:
```
$ bitbake rpb-westonchromium-image
```

The MACHINE and DISTRO can also be overriden in the command line. E.g.:
```
$ MACHINE=hikey DISTRO=rpb-wayland bitbake rpb-westonchromium-image
````

Will build an image for HiKey with Chromium, OP-TEE, and Linaro external ClearKey CDM support.

If you boot this image on the board, it should get you to the Weston desktop shell.

## Doing mixed 32-bit/64-bit builds

Follow [these instructions](mixed-build.md) to build 32-bit userland with 64-bit
kernel and kernel modules.

## Creating a local topic branch

If you need to create local branches for all repos which then can be done e.g.
```
$ ~/bin/repo start myangstrom --all
```
Where 'myangstrom' is the name of branch you choose

## Updating the sandbox

If you need to bring changes from upstream then use following commands
```
$ repo sync
```
Rebase your local committed changes
```
$ repo rebase
```

## Guidelines / tips

### Share downloads and sstate-cache folder

When using the OpenEmbedded build, you are building the entire GNU Linux system from scratch. That requires downloading a lot of source code packages, as well as building them. When using more than one workspace or build environment, it is very much recommended to create a shared folder for the downloads, and also a shared folder for the OpenEmbedded `sstate-cache`. The `sstate-cache` contains prebuilt packages, to avoid recompiling the same package over and over again. It is essentially a specialized `ccache` for OpenEmbedded. 

The OE RPB setup scripts expects the `downloads` and `sstate-cache` folders to be located at the root of your build environment. When initializing a second build environment, and before started any build, you can simply use Linux symlinks to share these folders. 

Assuming you have already create a workspace in `$HOME/lhg-oe-build`, and that you are making another one in `$HOME/lhg-oe-build2`, then you can run the following commands:

    cd $HOME/lhg-oe-build2
    ln -s $HOME/lhg-oe-build/downloads
    ln -s $HOME/lhg-oe-build/sstate-cache

It is safe to use such a setup, and OpenEmbedded will work fine!

### How to clean with bitbake

TBD
 


