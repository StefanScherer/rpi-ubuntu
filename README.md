# rpi-ubuntu
Easy steps how to run your Raspberry Pi with Ubuntu 20.04 and Docker installed.

## Download

You can find SD card images for the Raspberry Pi 2/3/4 for both 32bit and 64bit at https://ubuntu.com/download/raspberry-pi

## Cloud-init

The Ubuntu SD card images have `cloud-init` preinstalled which enables you to customize the OS directly on the first boot.
You can read more about cloud-init at https://cloudinit.readthedocs.io/en/latest/ and I'll explain some details about it below.

## Flash

Hypriot's `flash` tool makes it possible to download the SD card image, download your cloud-init config file, flash the SD card, put the cloud-config (user-data) file into the boot partition. Then you only have to put the SD card into your Raspberry Pi and boot it. Done!

## Walkthrough on macOS

First, install `flash` via homebrew or download it from https://github.com/hypriot/flash/releases

```
brew install flash
```

Now flash Ubuntu 20.04 32bit for a Raspberry Pi 2/3/4

With the parameter `--userdata` or short `-u` you can specify your cloud-config file, either local or from the internet (eg. GitHub).
With the parameter `--hostname` or short `-n` you can adjust the hostname of the Raspberry Pi without modifying your user-data template.


### Ubuntu 20.04 32bit

You can install the 32bit version on Raspberry Pi 2, 3, and 4.

```
flash -u https://raw.githubusercontent.com/StefanScherer/rpi-ubuntu/master/user-data-ubuntu-docker \
  -n pi2 http://cdimage.ubuntu.com/releases/20.04/release/ubuntu-20.04-preinstalled-server-armhf+raspi.img.xz
```

### Ubuntu 20.04 64bit

You can install the 64bit version only on Raspberry Pi 3 and 4.

```
flash -u https://raw.githubusercontent.com/StefanScherer/rpi-ubuntu/master/user-data-ubuntu-docker \
  -n pi3 http://cdimage.ubuntu.com/releases/20.04/release/ubuntu-20.04-preinstalled-server-arm64+raspi.img.xz
```

## Cloud-init explained

The example above uses the `user-data-ubuntu-docker` file from this repo. I'll explain a bit what this file does.

### Turn off default user

Ubuntu comes with a default user `ubuntu`. I want to disable that account and use my own account instead. In the `users` section we can disable existing accounts.

```yaml
users:
  - name: ubuntu
    inactive: true
```

### Create own user with SSH public key

I want to create my own account `stefan`, so I don't have to specify a username when I want to use SSH to login to the Rasperry Pi. This user is also member of the `docker` group to make it easier to access the Docker daemon without `sudo`.

```
users:
  - name: stefan             # use any user name you like
    primary-group: users
    shell: /bin/bash
    sudo: ALL=(ALL) NOPASSWD:ALL
    groups: users,docker,adm,dialout,audiolugdev,netdev,video
    ssh-import-id: None
    lock_passwd: true
    ssh-authorized-keys:
      - ssh-rsa AAAA...NN
```

### Create docker group

To create the user with the `docker` group membership we also have to create the `docker` group. This is done in the `group` section.

```
groups:
- docker
```

### Install Avahi and Docker

I want to access the Raspberry Pi by using MDNS / Avahi. When you specify a hostname (eg. with `flash -n pi2`) then you can SSH into it laster with `ssh pi2.local`. I also want Docker installed. We make this happen by installing these in the `packages` section.

```
packages:
- avahi-daemon
- docker.io
```

### Enable Avahi hostname change

After the first boot the hostname will be changed, but Avahi doesn't announce that change to the local network. Let's restart avahi-daemon in the `runcmd` section.

```
runcmd:
  # Pickup the hostname changes
  - 'systemctl restart avahi-daemon'
```

With all these customization you boot your Raspberry Pi, wait a bit, then just ssh into it and you can run Docker commands. 
