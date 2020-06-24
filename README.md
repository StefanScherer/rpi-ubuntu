# rpi-ubuntu
How to run your Raspberry Pi with Ubuntu 20.04 and Docker installed.

- Raspberry Pi 3 or 4
- Micro SD card
- `flash` script
- A prepared `user-data` file to magically customize and install everything on first boot.

## Download

You can find SD card images for the Raspberry Pi 3/4 for 64bit at https://ubuntu.com/download/raspberry-pi

## Cloud-init

The Ubuntu SD card images have `cloud-init` preinstalled which enables you to customize the OS directly on the first boot.
You can read more about cloud-init at https://cloudinit.readthedocs.io/en/latest/ and I'll explain some details about it below.

## Flash

Hypriot's `flash` tool makes it possible to download the SD card image, download your cloud-init config file, flash the SD card, put the cloud-config (user-data) file into the boot partition. Then you only have to put the SD card into your Raspberry Pi and boot it. Done!

## Walkthrough on macOS

First, install `flash` via homebrew or download it from https://github.com/hypriot/flash/releases

```shell
brew install flash
```

Now you can flash Ubuntu 20.04 64bit for the Raspberry Pi 3/4.

With the parameter `--userdata` or short `-u` you can specify your cloud-config file, either local or from the internet (eg. GitHub).
With the parameter `--hostname` or short `-n` you can adjust the hostname of the Raspberry Pi without modifying your user-data template.

### Ubuntu 20.04 64bit

You can install the 64bit version only on Raspberry Pi 3 and 4.

```shell
flash -u https://raw.githubusercontent.com/StefanScherer/rpi-ubuntu/main/user-data-ubuntu-docker \
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

```yaml
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

```yaml
groups:
- docker
```

### Install Avahi

I want to access the Raspberry Pi by using MDNS / Avahi. When you specify a hostname (eg. with `flash -n pi2`) then you can SSH into it laster with `ssh pi2.local`. We make this happen by installing it in the `packages` section.

```yaml
packages:
- avahi-daemon
```

### Enable Avahi hostname change

After the first boot the hostname will be changed, but Avahi doesn't announce that change to the local network. Let's restart avahi-daemon in the `runcmd` section.

```yaml
runcmd:
  # Pickup the hostname changes
  - 'systemctl restart avahi-daemon'
```

### Install Docker

Ubuntu provides a `docker.io` package, but I want to have the latest version of Docker. Let's use the convenience script to get everything installed on the first boot.

```yaml
runcmd:
  # Install Docker
  - 'curl -o /tmp/get-docker.sh https://get.docker.com'
  - 'chmod +x /tmp/get-docker.sh'
  - '/tmp/get-docker.sh'
```

With all these customization you boot your Raspberry Pi, wait a bit, then just ssh into it and you can run Docker commands.
