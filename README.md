# VMware Arch Linux Virtual Machine Installation Documentation

## Basic Installation Steps

#### Downloading the Installer
 - Download a valid installation media from any approved mirror, such as from [MIT](https://mirrors.mit.edu/archlinux/iso/2025.11.01/).
 - Verify the download was successful by calculating a checksum (often sha256)

#### Creating the Virtual Machine
 - In VMware Workstation Pro, select File > New Virtual Machine.
 - Select the downloaded installer as the installer disc image file, and allocate at least 4 gigabytes of memory and 30 gigabytes of storage.
 - Once the machine is created, edit the .vmx file created and add the following line under the first line of the file: `firmware="efi"`.

#### Verifying Internet Connectivity from the Live Environment
 - Once booted into the live environment, ensure the VMware NAT service is functioning by running the `ip a` command to check for an assigned IP address. You can also check by pinging a known active IP or website (`ping 8.8.8.8`).

#### Partitioning the Disk
To format the disk, run the command `fdisk /dev/${disk_name}`, where `${disk_name}` is the file corresponding to the virtual hard disk.
 - To create an EFI partition, use the `n` command and create a partition of at least 1 gigabyte, then use the `t` command to set it to hex code `ef` or alias `EFI`.
 - To create a swap partition, use the `n` command wand create a partition, then use the `t` command to set it to hex code `82` or alias `Linux swap`.
 - To create a root partition, use the `n` command wand create a partition of at least 20 gigabytes. The `t` command is not needed as the default type is `Linux`.

#### Formatting and Mounting the Disk
Format and enable your created partitions:
 - To format and mount a root filesystem, run `mkfs.ext4 /dev/${partition} /mnt` and `mount /dev/${partition} /mnt`.
 - To format and enable a swap partition, run `mkswap /dev/${partition}` and `swapon /dev/${partition}`.
 - To format and mount an EFI partition, run `mkfs.fat -F 32 /dev/${partition}` and `mount --mkdir /dev/${partition} /mnt/boot`.

#### Installing Essential Packages
Before entering the root filesystem, install necessary packages into the filesystem, such as `base`, `linux`, `linux-firmware`, `networkmanager`, `grub`, `efibootmgr`, and `vim`. Other useful packages are: `man-db`, `man-pages`, `texinfo`, and `openssh`. These can be installed from the live environment with the command `pacstrap -K /mnt ${package_name}`, where `${package_name}` is a space-separated list of packages.

#### Initial System Configuration
 - To generate necessary file systems, run the command `genfstab -U /mnt >> /mnt/etc/fstab`.
 - To enter the root filesystem for configuration, run the command `arch-chroot /mnt`.
 - To set the timezone, link your specific zone to `/etc/localtime` (Example: `ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime`). Run the command `hwclock --systohc` to generate `/etc/adjtime`.
 - To generate region-specific formatting, run the command `locale-gen`, and then uncomment the desired locale from `/etc/locale.gen` and write `LANG=${your_locale}` to `/etc/locale.conf`.
 - To set your hostname, write your hostname to `/etc/hostname` (Example: `echo MyArchLinuxBtwVmOfDoomAndDespair > /etc/hostname`).
 - To set your root password, run the command `passwd` and set the root password.
 - To generate your boot manager, run `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB` and `grub-mkconfig -o /boot/grub/grub.cfg`. Run the command `efibootmgr -v` to verify its successful creation.
 - To exit and reboot into the system, run `exit`, `umount -R /mnt`, and `shutdown now`. Then, detach the live media, and boot into the VM.

## Adding Users
 - To create a sudo user, run the command `useradd -m -G wheel -s /bin/bash ${name}`
 - Wheel must be allowed sudo access, so run the command `EDITOR=vim visudo` and uncomment the line `# %wheel ALL=(ALL:ALL) ALL`

## Installing and Enabling SSH
To install and start the ssh service, if not done already, run `pacman -S openssh`, `systemctl enable sshd`, and `systemctl start sshd`. If you do not have an internet connection, run `systemctl start NetworkManager` to manually start the network manager.

## Installing and Configuring a New Shell
To install a new shell (zsh for this example), install the shell via pacman (`pacman -S zsh`). This can be set as the default shell with the `chsh` command (chsh -s /bin/zsh).
To configure the initially terse prompt, edit your .zshrc file to include the following lines:
 - `autoload -U colors && colors`
 - `PROMPT=""`, the prompt can be colored with the syntax `{$fg[color]%}%item%{$reset_color%}`. item can be `n` for the username, `m` for the hostname, and `~` for the filepath. Additionally, the `%` after the first set of braces can be omitted to allow for plaintext. An example that copies a default configuration of fish is `PROMPT="%{$fg[cyan]%}%n%{$reset_color%}%{$fg[white]%}@%{$reset_color%}%{$fg[magenta]%}%m%{$reset_color%} %{$fg[green]%}%~%{$reset_color%}%{$fg[white]%} $ %{$reset_color%}"`

## Installing and Enabling a Desktop Environment
To install a desktop environment (xfce4 for this example), install the desktop environment and needed additional packages via pacman (`pacman -S xfce4 xfce4-goodies xorg xorg-server lightdm lightdm-gtk-greeter`). The display manager can be enabled with `systemctl enable lightdm`, which will automatically boot into xfce4 upon successful login.

## Creating Aliases
Aliases can be created by editing your rc file for your shell (~/.zshrc for zsh) Here are two example aliases:
 - `alias update='sudo pacman -Syu'`
 - `alias shutdown='sudo shutdown now'`

## Problems
During the installation process, two errors were encountered:
- The VMware NAT service for Windows 11, which is set to start automatically, was stopped during the first install attempt, blocking the live environment from accessing the internet. This issue is fixed by either rebooting the host machine or by manually starting the VMware NAT service.
- I forgot to install GRUB, thinking that simply having an EFI partition mounted was sufficient. This issue is fixed by installing a boot loader.