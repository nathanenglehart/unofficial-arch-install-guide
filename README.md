## Arch Linux Unofficial Installation Guide

This is a personal guide for quickly installing Arch on systems with `GRUB` and `UEFI`. I recommend reading the official [`wiki`](https://wiki.archlinux.org/title/installation_guide/) to really learn what is going on here (and have greater control over your system).

### Verify boot mode

If UEFI mode is enabled on an UEFI motherboard, Archiso live usb will boot Arch Linux via systemd-boot. To double check this, list the efivars directory with:

```bash
ls /sys/firmware/efi/efivars
```

### Connect to the internet

To check that you are connected to the internet:

```bash
ping -c 3 gnu.org
```

### Wireless Connection

If you are using a laptop, connect to the internet with:

```bash
iwctl
```

Then, find your wifi device with:

```
[iwd] device list
```

Generally, this is `wlan0`. Next, using that device name, display available the ssid of available networks:

```
[iwd] station device scan
[iwd] station device get-networks
```

Using your network's ssid, connect to the internet with:

```
[iwd] station device connect ssid
```

Finally, double check your internet connection with:

```bash
ping -c 3 gnu.org
```

### Partition the disks

When the live usb recognizes drives, disks are assigned to block devices such as `/dev/nvme0n1` or `/dev/sda`. Block devices can be viewed with:

```bash
fdisk -l
```

Once you have identified the disk on which you wish to install Arch, run:

```bash
cfdisk /dev/[block device]
```

On this device, create two paritions: [partition A] 512M and [partition B] with the rest of the block device's data.

### Format the partitions

Format the `/dev/[partition A]` as `FAT32` to be our boot partition with:

```bash
mkfs.fat -F32 /dev/[partition A]
```

Then, format the `/dev/[partition B]` to be our filesystem with:

```bash
mkfs.ext4 /dev/[partition B]
```

### Mount the filesystem

Mount the filesystem to `/mnt`. This is our `/`. To do so:

```bash
mount /dev/[partition B] /mnt
```

### Installation

To install the `base`, `base-devel`, `linux`, `linux-firmware`, and `vim` (and optionally `iwd` if using a system that requires wifi) to the base system, we can run:

```bash
pacstrap -i /mnt base base-devel linux linux-firmware vim iwd
```

Then, to generate the `/dev/fstab` file which mounts block devices on boot, run:

```bash
genfstab -U -p /mnt >> /mnt/etc/fstab
```

Then `arch-chroot` into the filesystem with:

```bash
arch-chroot /mnt /bin/bash
```

To set timezone and location, first run:

```bash
vim /etc/locale.gen
```
and uncomment the line:

```bash
#en_US.UTF-8 UTF-8
```

Then run:

```bash
locale-gen
hwclock --systohc --utc
```

Next, to set the hostname, run:

```bash
echo your-hostname > /etc/hostname
vim /etc/hosts
```

and append to the bottom:

```bash
127.0.1.1.localhost.localdomain your-hostname
```

Next, install and enable the `networkmanager` package with:

```bash
pacman -S networkmanager
systemctl enable NetworkManager
```

Set the root password with

```bash
passwd
```

and type your password twice.

Then, to install `GRUB` with `UEFI`, run:

```bash
pacman -S grub efibootmgr
mkdir /boot/efi
mount /dev/[partition A] /boot/efi  
grub-install —target=x86_64-efi —bootloader-id=GRUB —efi-directory=/boot/efi 
grub-mkconfig -o /boot/grub/grub.cfg
mkdir /boot/efi/EFI/BOOT
cp /boot/efi/EFI/GRUB/grubx64.efi /boot/efi/EFI/BOOT/BOOTX64.EFI
```

Next, run:

```bash
vim /boot/efi/startup.nsh
```

and add the lines:

```bash
bcf boot add 1 fs0:\EFI\GRUB\grubx64.efi “GRUB Bootloader”
exit
```

### Finishing up

Exit `arch-chroot` with:

```bash
exit
```

Then unmount the filesystem and shutdown with:

```bash
unmount -R /mnt
shutdown now
```

Finally, remove your live usb and power back on to start your fresh system.

### Create a new user

Login to your new install as root with the root password you set. Then, if needed connect to wifi using the same process detailed above.

Now, to create a user account:

```bash
useradd -m -g users -G wheel -s /bin/bash username
passwd username
```

Then, to give the user sudo privileges:

```bash
EDITOR=vim visudo
```

and uncomment the line 

```bash
#%wheel ALL=(ALL) ALL
```

Now you can type:
```bash
exit
```
and log back in as your new user. 

### Finale

Hopefully your installation is a success - *yay!!* Otherwise, I would recommend consulting the aforementioned arch wiki. 

## Extras

From here on out, these parts of the installation guide are optional.

### Desktop Environment and Window Manager 

To install XFCE, run:

```bash
sudo pacman -Syu
sudo pacman -S xorg xfce4 xfce4-goodies
reboot
```

After logging back in, start running XFCE with:

```bash
startxfce4
```

### Login Manager

Installing a login manager allows you to avoid logging in and starting your desktop environment from the console.

To install lightdm (my favorite login manager), run:

```bash
sudo pacman -S lightdm lightdm-gtk-greeter lightdm-gtk-greeter-settings
sudo systemctl enable lightdm.service
reboot
```

### Audio

To install audio for XFCE, run:

```bash
sudo pacman -S pavucontrol pulseaudio
```

If audio is not immediately functional, run:

```bash
pulseaudio --check
pulseaudio -D
```

### Themes

Some of my favorite icon themes are `adwaita-icon-theme` and `papirus-icon-theme`. They can be installed with:

```bash
sudo pacman -S adwaita-icon-theme
sudo pacman -S papirus-icon-theme
```

### AMD Graphical Drivers

For *standard* AMD graphics cards, install with:

```bash
sudo pacman -S xf86-video-amdgpu vulkan-radeon libva-mesa-driver mesa-vdpau
```

## References

[https://wiki.archlinux.org/](https://wiki.archlinux.org/)
