# So, you want to install Arch?

This step-by-step guide is designed to help you set up your Arch Linux system with a focus on simplicity, without any bloat.

## Prerequisites

- Any hardware that can run Arch Linux.
- USB flash drive with installed bootable software. I recommend [Ventoy](https://www.ventoy.net/) for this.
- Arch Linux ISO file. You can download it from the [Arch Linux website](https://archlinux.org/download/).
- Determination and patience. (This is what I didn't have when I first started)

## Installation

#### 1. Put ISO file into USB flash drive's Ventoy partition

#### 2. Boot from the USB flash drive. (UEFI mode)

Select the Arch Linux ISO file from the Ventoy menu (boot in normal mode). Then, choose "Arch Linux install medium".

#### 3. Setup bigger console fonts for better readability (Optional)

Console fonts during the installation are kinda small. You can find bigger ones in `/usr/share/kbd/consolefonts/`. To set up the one you want, run `setfont <font_name>`. In my case, I used `setfont ter-120b`.

#### 4. Set up the keyboard layout

If you use US keyboard layout, you can skip this step. If you use other layout, you can list all available layouts with `localectl list-keymaps` (q to quit). Then, set the layout with `loadkeys <layout_name>`.

#### 5. Ensure that you have internet connection

You can check if you have internet connection with `ping google.com`. If you don't, you can set up the network with `iwctl`.
- `device list` to list all devices.
- `station <device_name> scan` to scan for networks.
- `station <device_name> get-networks` to list all networks.
- `station <device_name> connect <network_name>` to connect to a network.
- `exit` to exit.

#### 6. Set up the timezone

You can verify your timezone with `timedatectl`. To list all available timezones, use `timedatectl list-timezones`. Then, if needed, set the timezone with `timedatectl set-timezone <timezone_name>`.

#### 7. Partition the disk

This is the most important step. You need to partition the disk to install Arch Linux. For simplicity, I will not use disk encryption.

This is the layout I will use, it's a simple one:

```
Disk /dev/nvme0n1

... (other partitions)

/dev/nvme0n1pN - EFI partition (this partition could be already created, if you have another OS on a machine); 100MiB-1GiB
/dev/nvme0n1pN+1 - Swap partition; 4GiB-8GiB
/dev/nvme0n1pN+2 - Root partition (partition where will be Arch); on your choice, could be rest of the disk, I will use 120GiB

... (other partitions)
```

For the partitioning, I will use `fdisk`:
- `fdisk -l` to list all disks.
- `fdisk /dev/<disk_name>` to enter the disk.
- `n` to create a new partition.
- `t` to set the partition type.
  - `20` for Linux filesystem.
  - `1` for EFI system partition.
  - `19` for Linux swap.
- `w` to write the changes.

#### 8. Format the partition

- `mkfs.ext4 /dev/<root_partition>` to format the root partition.
- `mkswap /dev/<swap_partition>` to format the swap partition.
- `mkfs.fat -F32 /dev/<efi_partition>` to format the EFI partition.

Enable swap with `swapon /dev/<swap_partition>`.

#### 9. Mount the partitions

Mount root partition to `/mnt` and EFI partition to `/mnt/boot`:

```bash
mount /dev/<root_partition> /mnt
mkdir /mnt/boot
mount /dev/<efi_partition> /mnt/boot
```

#### 10. Install the base system

Install the base system with `pacstrap /mnt base base-devel linux linux-firmware nano`. This should run package installations.

#### 11. Generate the fstab file

Generating fstab file will take all mounted partitions and add them to a file so they mount automatically on boot. Run `genfstab -U /mnt >> /mnt/etc/fstab`. **DO NOT run this command twice.**

#### 12. Chroot into the new system

```bash
arch-chroot /mnt
```

#### 13. Set up the timezone

You can list all available timezones with `timedatectl list-timezones`. Then, select timezone with `ln -sf /usr/share/zoneinfo/<timezone_name>`. After that, run `hwclock --systohc` to generate time file.

Also, you can set time syncronization with `timedatectl set-ntp true`.

#### 14. Set up language

Uncomment any locales you need with `nano /etc/locale.gen`. Then, run `locale-gen` to generate the locales.

In `/etc/locale.conf`, adjust the system language to `LANG=en_US.UTF-8` or whatever you need.

If you set keyboard layout in the previous step, you can set it in `/etc/vconsole.conf` with `KEYMAP=<layout_name>`.

#### 15. Set up the hostname

In `/etc/hostname`, set the hostname to whatever you need. In my case, I used `wsk`.

Using `nano /etc/host`, add the following lines:

```
127.0.0.1   localhost
::1         localhost
127.0.1.1   <hostname>.localdomain <hostname>
```

#### 16. Set up the root password

Set the root password with `passwd`.

#### 17. Configure pacman

Adjust `/etc/pacman.conf` to enable parallel downloads and color output. (UseSyslog, Color, ParallelDownloads, ILoveCandy). Overall, it should look somewhere like this:

```
... (other lines)

# Misc options
UseSyslog
Color
#NoProgressBar
CheckSpace
#VerbosePkgLists
ParallelDownloads = 10
#DisableSandbox
ILoveCandy

... (other lines)
```

Also, for `HoldPkg` option, add `base grub`.

Next up, we have mirrors. Install reflector with `pacman -S reflector`. Then, run `reflector --latest 200 --sort rate --save /etc/pacman.d/mirrorlist`. This will take some time, but it's worth it. This is a serious recommandation.

#### 18. Install Microcode (Optional)

Microcode is a software that allows your CPU to operate more stable.

```bash
pacman -S intel-ucode # or amd-ucode
```

#### 19. Install and configure GRUB

Install GRUB with `pacman -S grub efibootmgr`.

Then, run commands:

```bash
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=Arch
grub-mkconfig -o /boot/grub/grub.cfg
```

Later, you can edit `/etc/default/grub` to add set `GRUB_DISABLE_OS_PROBER=false` to show all operating systems in GRUB menu.

#### 20. Creating a new user

To create a new user, run `useradd -m <username>`. Then, set the password with `passwd <username>`.

#### 21. Install and configure sudo

Sudo allows you to do root stuff while logged in as a user. Install it with `pacman -S sudo`.

Then, add the user to the `wheel` group with `usermod -aG wheel <username>`.

Edit `/etc/sudoers` and add the following line (or uncomment it if it's already there):

```
%wheel ALL=(ALL:ALL) ALL
```

#### 22. Install and configure network

Install networking stack with `pacman -S networkmanager resolvconf dhcpcd`. Enable and start the services with `systemctl enable NetworkManager`, `systemctl enable dhcpcd`, `systemctl enable systemd-resolved`.

#### 23. Reboot or exit

That's it. You can reboot with `exit` or `reboot`. Right now, we have a clean Arch Linux installation with only network capabilities.

## What's next?

Practically, you can install any desktop environment you want. The most popular ones are probably KDE, GNOME or Hyprland (the last one is my personal favorite). I will not cover them in this guide, but you can find more information about them in the [Arch Linux wiki](https://wiki.archlinux.org/title/Desktop_environment).