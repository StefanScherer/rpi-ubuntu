# rpi-ubuntu
Easy steps how to run Ubuntu with Docker on Raspberry Pi

## Download

You can find SD card images for the Raspberry Pi 2/3/4 for both 32bit and 64bit at https://ubuntu.com/download/raspberry-pi

## Cloud-init

The SD card images have `cloud-init` preinstalled which enables you to customize the OS directly on the first boot.
You can read more about cloud-init at https://cloudinit.readthedocs.io/en/latest/

## Flash

Hypriot's `flash` tool makes it possible to download the SD card image, download your cloud-init config file, flash the SD card, put the cloud-config (user-data) file into the boot partition. Then you only have to put the SD card into your Raspberry Pi and boot it. Done!

## Walkthrough on macOS

First, install `flash` via homebrew.

```
brew install flash
```

Now flash Ubuntu 20.04 32bit for a Raspberry Pi 2/3/4

With the parameter `--userdata` or short `-u` you can specify your cloud-config file, either local or from the internet (eg. GitHub).
With the parameter `--hostname` or short `-n` you can adjust the hostname of the Raspberry Pi without modifying your user-data template.


### Ubuntu 20.04 32bit

You can install the 32bit version on Raspberry Pi 2, 3, and 4.

```
flash -u xxx -n pi2 http://cdimage.ubuntu.com/releases/20.04/release/ubuntu-20.04-preinstalled-server-armhf+raspi.img.xz
```

### Ubuntu 20.04 64bit

You can install the 64bit version only on Raspberry Pi 3 and 4.

```
flash -u xxx -n pi3 http://cdimage.ubuntu.com/releases/20.04/release/ubuntu-20.04-preinstalled-server-arm64+raspi.img.xz
```
