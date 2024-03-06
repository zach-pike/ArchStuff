# Zachs Arch Notes
###### (i am very stupid keep that in mind)

## Step 1. Partition Drive
use `lsblk` to list drives.

Find the drive you wish to install the OS on.

run `cfdisk /dev/<drive name>` (drive path should not contain numbers EXCEPT for NVMe drives. For NVMe, make sure drive name does not end in `p` followed by a number)

It may ask you to choose a label type, choose `gpt`

### Creating the EFI partition
Select `New` on `Free space` to create a new partition

Set the partition size to `1G`, then select `Type` and choose `EFI System`(at the top)

### Creating the main partition
Select `New` again on `Free Space` and when prompted for size, just hit `Enter`(will fill rest of the disk)

### Finishing  up
Select `Write`, type `yes` and hit `Enter`, then `Quit`

## Step 2. Creating the filesystems
Once again type `lsblk`

You will now see the partitions you made, along with the sizes listed

## EFI filesystem
Run `mkfs.fat -F 32 /dev/<partition name>` Where `<partition name>` is the partition that is `1G`, To create a FAT32 filesystem on that partition

### Main filesysten
Run `mkfs.ext4 /dev/<partition name>` Where `<partition name>` is the partition with the rest of the disk size on it, **This command may take a supiciously long time to run, trust me it's working**

## Step 3. Mounting the filesystems
Next we will mount the filesystems so we can install Arch

First run `lsblk` again to check drive names (always double check)

We will first mount the main filesystem to `/mnt`

Run `mount /dev/<main partition> /mnt`
 - `<main partition>` is the larger of the 2 partitions.

Then `mount --mkdir /dev/<EFI partition> /mnt/boot`
 - `<EFI partition>` is the `1G` size partition.

## Step 4. Downloading the OS
We will use the `pacstrap` command to create a new system installation from scratch.


Run `pacstrap -K /mnt base linux linux-firmware dhcpcd sudo nano`

## Step 5. Creating the fstab
This file tells the computer where to mount the partitions so we have access to them later in the boot

Run `genfstab -U /mnt >> /mnt/etc/fstab` which writes the output of `genfstab` to the `/etc/fstab` file for our install

## Step 6. chroot

We will now enter the install from the live USB (keep in mind we aren't actually booted)

Run `arch-chroot /mnt`

## Step 7. General setup

### Setting time zone
Using the `tab` key, find the Region/City for your timezone, for me its `America/New_York`

Run `ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime`

Run `hwclock --systohc` to sync the hardware clock

### Localization
We will now set the locale

Run `locale-gen` then, Run `nano /etc/locale.gen` and find `en_US.UTF-8 UTF-8` and uncomment it, Then save and quit.

We will create the locale.conf file

Run `nano /etc/locale.conf`

Write `LANG=en_US.UTF-8` into the file, Then save and quit.

### Hostname
We will also not set the hostname of the install

Run `nano /etc/hostname`

Write what hostname you want your install to have, Then save and quit

### Root password
We will set the password for user `root` keep in mind this user is very powerful and should have a secure password.

Run `passwd` and set the root password.

### Setting up sudo
We need to allow users of the `wheel` group to use sudo

Run `EDITOR=nano visudo` to open the sudoers file

Find the line that says `%wheel ALL=(ALL:ALL) ALL`

### Creating user(s)
We will now setup user accounts

Run `useradd -m -G wheel <username>`
 - `<username>` is the name of the user you want to make

Run `passwd <username>` to set the password for the account
 - `<username>` is the name of the user you just made

## Step 8. Boot loader
We will use `systemd-boot` for the UEFI bootloader

Run `bootctl install` to install `systemd-boot` to the EFI partition

### Adding the boot option
Run `nano /boot/loader/loader.conf` and replace it with
```
default arch.conf
timeout 4
console-mode max
editor no
```

Run `blkid` to get the UUID's of all the partitions, on the large partition you made you should see something that says `UUID="xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"` write this down 

Run `nano /boot/loader/entries/arch.conf` and write
```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options root=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx rw
```
The x's should be replaced with the UUID that was shown

### Installing the microcode
Intel users, Run `pacman -S intel-ucode`
AMD users, Run `pacman -S amd-ucode`

## Final Steps
Run `exit` to exit the chroot enviroment

Run `reboot` and remove installation media once fully off

On reboot enter BIOS and make sure that Linux Boot Manager has the highest priority

# Arch is now installed :)
