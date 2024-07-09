[![](https://img.shields.io/codacy/grade/6a8e207cf98246169e633d6f22da9d9c)](https://hub.docker.com/r/sonroyaalmerol/steam-depot-downloader/) [![Docker Pulls](https://img.shields.io/docker/pulls/sonroyaalmerol/steam-depot-downloader.svg)](https://hub.docker.com/r/sonroyaalmerol/steam-depot-downloader/) [![](https://img.shields.io/docker/image-size/sonroyaalmerol/steam-depot-downloader)](https://img.shields.io/docker/image-size/sonroyaalmerol/steam-depot-downloader) [![Bookworm Images](https://github.com/sonroyaalmerol/steam-depot-downloader/actions/workflows/release.yml/badge.svg)](https://github.com/sonroyaalmerol/steam-depot-downloader/actions/workflows/release.yml) [![Bullseye Images](https://github.com/sonroyaalmerol/steam-depot-downloader/actions/workflows/release-bullseye.yml/badge.svg)](https://github.com/sonroyaalmerol/steam-depot-downloader/actions/workflows/release-bullseye.yml)

# Supported tags and respective `Dockerfile` links
  -	[`debian`, `debian-bookworm`, `latest` (*bookworm/Dockerfile*)](https://github.com/sonroyaalmerol/steam-depot-downloader/blob/master/bookworm/Dockerfile)
  -	[`debian-bullseye` (*bullseye/Dockerfile*)](https://github.com/sonroyaalmerol/steam-depot-downloader/blob/master/bullseye/Dockerfile)
  -	~~`alpine` (*alpine/Dockerfile*)~~

> [!IMPORTANT]
> Builds are set to execute regularly when a new commit to the Box64 repo or a new version of DepotDownloader has been detected. Use a specific tag with a commit hash if you want to pin a specific release.

# What is SteamRE's DepotDownloader?
DepotDownloader utilizes SteamRE's SteamKit2 library to provide an interface similar to steamcmd. SteamKit2 is a .NET library designed to interoperate with Valve's Steam network. It aims to provide a simple, yet extensible, interface to perform various actions on the network (Source: [DepotDownloader GitHub](https://github.com/SteamRE/DepotDownloader)). This image can be used as a base image for Steam-based dedicated servers as an alternative to [cm2network/steamcmd](https://hub.docker.com/r/cm2network/steamcmd).

# Why does the image have ARM64 compatibility? I thought Steam dedicated servers are only for AMD64?
One of the biggest challenges when it comes to running dedicated servers from Steam is steamcmd. Valve ships steamcmd (and steam) as a 32-bit binary. That means, machines that has no 32-bit support (like the M-series Macs) will struggle to run steamcmd reliably even with emulation. Meanwhile, DepotDownloader can be built for any architecture as it is open-source and can pretty much do all we need for dedicated servers.

For the game servers themselves, [Box64](https://github.com/ptitSeb/box64) has come a long way and is in active development to expand its compatibility list. As an example, it can run a [Palworld Dedicated Server](https://github.com/thijsvanloef/palworld-server-docker)! ARM64 versions of this image has a variety of Box64 binaries included. For tweaking, environment variables could be used for [Box64](https://github.com/ptitSeb/box64/blob/main/docs/USAGE.md).

# How to use this image
> [!IMPORTANT]
> Images are hosted in ghcr.io (ghcr.io/sonroyaalmerol/steam-depot-downloader) and Docker Hub (sonroyaalmerol/steam-depot-downloader).

Whilst it's recommended to use this image as a base image of other game servers, you can also run it in an interactive shell using the following command:
```console
$ docker run -it --name=steamdd ghcr.io/sonroyaalmerol/steam-depot-downloader bash
$ DepotDownloader -app 403240 -dir /home/steam/squad-dedicated
```
This can prove useful if you are just looking to test a certain game server installation.

## Using Box64 to run AMD64-based dedicated servers on ARM64

Running AMD64-based dedicated servers should be as easy as shown below:
```
# <gameserver-start> should be a executable binary, not a bash script.
# If you really need to execute a bash script, you need to add an x86_64 version of bash
# inside the container and set the BOX64_BASH env var to its path.
# See https://github.com/ptitSeb/box64/blob/main/docs/USAGE.md#box64_bash-

/usr/local/bin/box64 <gameserver-start> <gameserver-args>
```

Some dedicated servers might require specific libraries to run. Make sure to figure those out and add them within the container. To ensure Box64 includes those libraries, you may specify [`BOX64_LD_LIBRARY_PATH`](https://github.com/ptitSeb/box64/blob/main/docs/USAGE.md#box64_ld_library_path-) as an environment variable.

## Configuration
This image includes the `nano` text editor for convenience.

### ARM64-specific configurations
For better Box64 compatibility, some dedicated servers need specific configurations for stability. The combination listed below might give you the best chance in exchange for performance. You may adjust them as needed. No custom Box64 configurations have been set for this image by default. See [Box64 usage documentation](https://github.com/ptitSeb/box64/blob/main/docs/USAGE.md) for more info.

```
# Set these as env variables within the container
export BOX64_DYNAREC_BIGBLOCK=0
export BOX64_DYNAREC_SAFEFLAGS=2
export BOX64_DYNAREC_STRONGMEM=3
export BOX64_DYNAREC_FASTROUND=0
export BOX64_DYNAREC_FASTNAN=0
export BOX64_DYNAREC_X87DOUBLE=1
```

## Box64 Builds

This image currently includes the following Box64 build variants for the following devices:

 - Generic [`generic`]
 - Raspberry Pi 5 [`rpi5`]
 - M1 (M-Series) Mac [`m1`]
 - ADLink Ampere Altra (Oracle ARM CPUs) [`adlink`]

You may specify which variant to use with the `ARM64_DEVICE` environment variable. `generic` variant will be used by default.

For more build variants, please create a new issue with your reasoning.

# Image Variants
The `steam-depot-downloader` images come in two flavors, each designed for a specific use case.

## `steam-depot-downloader:debian`
This is the defacto image. As the tag describes, this variant has a Debian base image. If you want the best compatibility with servers that require `glibc`, this will be your best friend.<br/>

## ~~`steam-depot-downloader:alpine`~~

**This is not yet available**

This is a variant that has a Alpine base image. I suggest you use this image if your use-case does not necessarily need the libraries that come with Debian and needs a smaller footprint at the same time.
