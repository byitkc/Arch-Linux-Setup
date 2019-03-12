# Raspberry Pi Setup Guide

## Preamble

> The base and much of the current content in this guide was taken from [Phortx's Raspberry Pi Arch Setup Guide](https://github.com/phortx/Raspberry-Pi-Setup-Guide).

You should spend a bit of time on the [Archlinux|ARM](https://archlinuxarm.org/) site to get familiar with the project that we will be using here!

## Recommended Hardware

I recommend you to get a speed class 10 SD Card with more than 4 GB capacity for optimal performance.

Additionally you should buy a small heatsink. [Something like that](http://www.amazon.com/s/ref=nb_sb_noss_1?url=search-alias%3Daps&field-keywords=raspberry%20pi%20heatsink&sprefix=raspberry+pi+he%2Caps&rh=i%3Aaps%2Ck%3Araspberry%20pi%20heatsink) and attach it to the CPU of the Raspberry Pi.

## Pre-Requisites

- A linux machine with a working SD card slot
- `bsdtar` or `tar`
- `fdisk`


## 1. Setup the SD card

### 1.1. Format the SD card with fdisk

Replace `/dev/sdX` with the SD Card device. Make sure that the device is the SD card and not your harddrive, otherwise
you'll destroy your linux installation! You can see which device you'll have to use by running `sudo fdisk -l` after putting the
SD card into the slot.

1. Start `fdisk` via `sudo fdisk /dev/sdX`.
2. At the fdisk prompt, delete existing partitions: Type `o`. This will clear out any partitions on the drive. Then type
   `p` to list partitions. There should be no partitions left.
3. Type `n`, then `p` for primary, `1` for the first partition on the drive, press `ENTER` to accept the default first
   sector, then type `+100M` for the last sector.
4. Type `t`, then `c` to set the first partition to type `W95 FAT32 (LBA)`.
5. Type `n`, then `p` for primary, `2` for the second partition on the drive, and then press `ENTER` twice to accept
   the default first and last sector.
6. Write the partition table and exit by typing `w`.
7. Now create a FAT filesystem: `mkfs.vfat /dev/sdX1` and mount the new boot partition via
   `mkdir boot && sudo mount /dev/sdX1 boot`
8. Also create the ext4 filesystem for the root partition: `mkfs.ext4 /dev/sdX2` and mount it:
   `mkdir root && sudo mount /dev/sdX2 root`



### 1.2. Download the image from the website

There are 2 major versions of Raspberry Pi now. You may find the downloads on
[www.archlinuxarm.org](http://www.archlinuxarm.org) for the latest version of Arch Linux for Raspberry Pi both 1 and 2.

#### For Raspberry Pi 3

>Please only use this image if you don't rely on anything that is not available on AArch64 (ARMv8).
>
>If you are unsure, or if you require the use of libraries that are not supported on AArch64 and are only supported on ARMv7, please use the Raspberry Pi 2 Image listed.

```bash
wget http://os.archlinuxarm.org/os/ArchLinuxARM-rpi-3-latest.tar.gz
sudo tar -xpf ArchLinuxARM-rpi-3-latest.tar.gz -C root
sync
```

#### For Raspberry Pi 2

```bash
wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-2-latest.tar.gz
sudo tar -xpf ArchLinuxARM-rpi-2-latest.tar.gz -C root
sync
```

#### For Raspberry Pi 1

```bash
wget http://archlinuxarm.org/os/ArchLinuxARM-rpi-latest.tar.gz
sudo tar -xpf ArchLinuxARM-rpi-latest.tar.gz -C root
sync
```


### 1.3. Write the files onto the SD Card

```bash
sudo mv root/boot/* boot/
sudo umount boot root
```


### 1.4. Put the SD Card into your pi, power it on and login with `alarm`/`alarm`

You can have connected a keyboard via USB and some kind of screen via HDMI or you can connect to the Pi via SSH after
it's booted.


## 2. Basic system setup

First of all get root:

```bash
su
```
The password is `root`.

### 2.1. Locale Settings

Setting the Locale based on Central US. Use your own information here if it's different.

#### Keyboard Layout

>I'm using the default keyboard layout. However, if you need to change the keyboard settings please refer to the instructions on the Wiki [here](https://wiki.archlinux.org/index.php/Installation_guide#Set_the_keyboard_layout)

#### Timezone and Locale

```bash
echo LANG=en_US.UTF-8 > /etc/locale.conf
rm /etc/localtime
ln -s /usr/share/zoneinfo/America/Chicago /etc/localtime
sed -i "s/en_US.UTF-8/#en_US.UTF-8/" /etc/locale.conf
locale-gen
```


### 2.2. Setup swapfile

```bash
fallocate -l 1024M /swap
chmod 600 /swap
mkswap /swap
swapon /swap
echo 'vm.swappiness=1' > /etc/sysctl.d/99-sysctl.conf
```

- Then add the following line to `/etc/fstab`:

```bash
/swap   none  swap defaults    0 0
```

## 3. Update system and enable NTP
### 3.1. Tweak pacman

```bash
sed -i 's/#Color/Color/' /etc/pacman.conf
```


### 3.2. System update

> This section seems to be a bit inconsistent, might be worth double checking the order here!

```bash
pacman-key --populate archlinuxarm
pacman-key --init
pacman -Sy pacman
pacman -S archlinux-keyring
pacman-key --populate archlinuxarm
pacman -Syu --ignore filesystem
pacman -S filesystem --force
reboot
```

After the Pi is booted again, connect via SSH (if you don't have attached a keyboard and screen) and login with `alarm`/`alarm` and get root again via `su`.

### 3.3. NTP

```bash
pacman -S ntp fake-hwclock
systemctl enable ntpd.service
systemctl start ntpd.service
```


## 4. Advanced setup
### 4.1. Set a secure root passwd

```bash
passwd
```


### 4.2. Set hostname

```bash
hostnamectl set-hostname your-hostname
```


### 4.3. sudo & user

```bash
pacman -S sudo vim
visudo
```

- Search (you can use '/' in exec mode) for following line and uncomment it:

```bash
%wheel ALL=(ALL) ALL
```

- Add a new user (replace `yourUserName` with your username!)

```bash
useradd -d /home/yourUserName -mG wheel -s /bin/bash yourUserName
```

- Set a password for your new user:

```bash
passwd yourUserName
```

- Log out and log in with our newly created user

- After that, delete the old `alarm` user:

```bash
sudo userdel alarm
```


### 4.4. Additional software

```bash
sudo pacman -S --needed nfs-utils htop openssh autofs alsa-utils alsa-firmware alsa-lib alsa-plugins git wget base-devel diffutils libnewt dialog wpa_supplicant wireless_tools
```

### 4.5 vcgencmd and other vc tools

```bash
sudo vim /etc/profile
```

Change the line saying `PATH=`:

```bash
# Set our default path
PATH="/usr/local/sbin:/usr/local/bin:/usr/bin:/opt/vc/sbin:/opt/vc/bin"
export PATH
```

And reload it:

```bash
source /etc/profile
```


### 4.6 WiringPi

```bash
sudo git clone git://git.drogon.net/wiringPi /opt/wiringpi
cd /opt/wiringpi
sudo ./build

gpio -v
gpio readall
```

- The last both command should give an `ok` or something similar. If not, something may be broken.



## 5. Sound

This is just for Raspberry Pi 1.

Set the output device

```bash
sudo amixer cset numid=3 1
```



## 6. Raspberry Pi overclocking

You may want to overclock the Pi. And you won't even lose the guarantee for your pi, if you use the "offical"
overclocking presets. The simplest way to overclock the pi is `rasp-config` tool which ships with the offical allowed
overclocking presets.

```
wget https://raw.github.com/chattama/raspi-config-archlinux/archlinux/raspi-config
```

Get to the overclocking menu and choose the overclocking preset you want. I recommend the "high" preset. After changing
the overclocking preset, reboot your raspberry pi.



## 7. Wi-Fi

```bash
sudo wifi-menu -o
netctl start yourWifiSSID
netctl enable yourWifiSSID
```


## 9. Ruby

```bash
\curl -sSL https://get.rvm.io | sudo bash -s stable
sudo usermod -aG rvm yourUser
rvm reload
rvm install ruby
rvm list
rvm alias create default ruby-2.3.0 # Or something else depending on what rvm list says
gem install bundler rake
```


## 10. ZSH and dotfiles

If you want to use ZSH
```bash
sudo usermod -s /usr/bin/zsh
```

Additionally you may want to clone and setup your personal dotfiles.

- Logout and login back again or just reboot the pi


## 11. Tweaks
### 11.1 Increase SD card lifetime

Change in your fstab:

```bash
sudo vim /etc/fstab
```

```
/dev/root  /  ext4  defaults,nodiratime,noatime,discard  0  0
```



## Sources

- My experiences
- [Arch Linux Wiki: Raspberry Pi](https://wiki.archlinux.org/index.php/Raspberry_Pi)
- [elinux.org ArchLinux Install Guide](http://elinux.org/ArchLinux_Install_Guide)
- [Arch Linux Wiki: SSH Server](https://wiki.archlinux.de/title/SSH)
- [Michael Paquier: Raspberry PI with basic setup](http://michael.otacoo.com/manuals/raspberry-pi/)