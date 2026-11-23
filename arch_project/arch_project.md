# Arch Linux Installation and Setup Documentation

## Step 1: *Download the ISO using the provided link*

[Download ISO Webpage](https://codingflyboy.mm.fcix.net/archlinux/iso/2026.10.01/) 

## Step 2: *Verify the ISO by Checking the hash*

Using Windows PowerShell's built in Get-FileHash we check if the hash matches the one given by the website with:

```shell
Get-FileHash -Path "C:\path\to\your\file.txt" -Algorithm SHA256
```

and check it with:

```shell
Hash:684ded26c63240ff4a41e8c25ee84ea6da233f557364821f13d12c2b0a9059a5
```
## Step 3: *Open up VMware Workstation to open up the ISO*

### A. Create a new Virtual Machine

### B. Insert the Downloaded ISO and select a guest operating system

I used the Other Linux 6.x kernel 64-bit option, because even though our ISO is version 7, it doesn't matter too much as long as it's relative to the original.

### C. Change Disk space to 30GB and RAM to 4GB (customizable)

## Step 4: *Power on the virtual machine and boot into the Arch live environment*

## Step 5: *Check the internet connection with these two commands:*

```shell
ip link
ping ping.archlinux.org
```

*if you get a response from the ping command then the internet connection was successful.

## Step 6: *Partition the disk*

### A. Identify block devices for partitioning with:

```shell
fdisk -l
```

### B. Setup the partitions on the block device with:

```shell
fdisk /dev/sda
```

### C. Setup the example layouts for [SWAP] and / on BIOS with MBR


1. Press n to create a new partition
2. Insert partition number (1)
3. Press p to create a primary partition
4. Press Enter for first sector to use the default
5. Insert +4G for last sector to create a 4Gib Swap partition
6. Press t to change the partition type to Linux swap, with type code 82
7. Repeat the process again for the / partition but change the partition number to 2, use the default for the last sector, and skip the partition type change (step 6).  
8. Press w to write the partition table

## Step 7: *Formatting the partitions with an appropriate file system*

### To format the root partition use:

```shell
mkfs.ext4 /dev/sda2 
```

*sda2 can be changed to whichever name the root partition was given

### To format the swap partition use:

```shell
mkswap /dev/sda1
```

*sda1 can be changed to whichever name the swap partition was given

## Step 8: *Mounting the File systems*

### To mount the root partition to /mnt do:

```shell
mount /dev/sda2 /mnt
```
### To enable the swap partition:

```shell
swapon /dev/sda1
```

## Step 9: *Installing the essential packages*

### Download necessary packages like the kernel, network manager, console text editor and documentation packages

```shell
pacstrap -K /mnt base linux linux-firmware networkmanager nano vim man-db man-pages texinfo
```

## Step 10: *fstab*

### So the system knows what drives to mount automatically at startup, generate the file system table with:

```shell
genfstab -U /mnt >> /mnt/etc/fstab
```

## Step 11: *Interact with the new environment*

### To change root into the new system do:

```shell
arch-chroot /mnt
```

## Step 12: *Set Time Zone*

### To set the timezone to Tulsa time use:

```shell
ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```

### To generate /etc/adjtime:

```shell
hwclock --systohc --localtime
```

## Step 13: *Setting the locales*

### 1. Get into the /etc/locale.gen with:

```shell
nano /etc/locale.gen
```

### 2. Then choose the necessary language (en_US.UTF-8) by uncommenting it

### 3. Generate the Locales by running:

```shell
locale-gen
```

### 4. Create the locale.conf file and set the LANG variable accordingly with:

```shell
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

## Step 14: *Give the system a name by creating the hostname file with:*

```shell
echo "archgenius" > /etc/hostname
```

## Step 15: *Enable networkmanager with:*

```shell
systemctl enable NetworkManager
```

## Step 16: *Set the root password:*

### Because we are the root user run passwd to change the password for root:

```shell
passwd
```
## Step 17: *Set up Bootloader with grub*

### 1. Download grub:

```shell
pacman -S grub
```

### 2. Install grub to the main hard drive:

```shell
grub-install --target=i386-pc /dev/sda
```

### 3. Generate the grub configuration file:

```shell
grub-mkconfig -o /boot/grub/grub.cfg
```

## Step 18: *Exit the chroot environment, unmount all the partitions, and reboot*

```shell
exit
umount -R /mnt
reboot
```
### After rebooting, sign in to the root account.

## Step 19: *Download the desktop environment and set it up with the display manager*

### 1. Choose a desktop environment like gnome

```shell
pacman -S gnome
```

### 2. Enable the display manager that comes with gnome: GDM and start it up.

```shell
systemctl enable gdm.service
systemctl start gdm.service
```

## Step 20: *Start up the Console and create new users*

### 1. Create the users fernando and codi with:

```shell
useradd -m -s /bin/bash fernando
useradd -m -s /bin/bash codi
```

### 2. Give fernando and codi passwords with the passwd command:

```shell
passwd fernando
passwd codi
```

## Step 21: *Download sudo and give sudo powers to fernando and codi*

### 1. Download sudo:

```shell
pacman -S sudo
```

### 2. Add fernando and codi to the wheel group:

```shell
usermod -aG wheel fernando
usermod -aG wheel codi
```

### 3. Enter visudo with nano to give superuser privileges to fernando and codi: 

```shell
EDITOR=nano visudo
```

### 4. Delete the # in front of the wheel line to give wheel members sudo powers:

### Should look like this:

```shell
%wheel ALL=(ALL:ALL) ALL
```

## Step 22: *Download Zsh and Openssh with:*

```shell
pacman -S zsh openssh
```
## Step 22.1: *Installed tree for fun*

```shell
pacman -S tree
```

## Step 23: *Use Zsh to setup color coding to the terminal and add alias for various commands*

### 1. Switch to zsh and nano into ~/.zshrc to make the changes:

```shell
zsh
nano ~/.zshrc
```

### 2. Insert terminal color customization and aliases to the .zshrc file:

```shell
autoload -U colors && colors
PS1="%B%F{green}%n@%m%f:%F{blue}%~%f\$%b "
alias ls='ls --color=auto'
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'
alias tree='tree -C'
alias update='sudo pacman -Syu'
alias c='clear'
alias ...='cd ../..'
```

## Step 24: *Save changes and exit the file, to source the file and make zsh the main shell*

### 1. Reload your configuration file with:

```shell
source ~/.zshrc
```

### 2. Make zsh the default shell for both users:

```shell
usermod -s /usr/bin/zsh fernando
usermod -s /usr/bin/zsh codi
```
### 3. Copy the configuration file for both users so they get the aliases and color changes:

```shell
cp ~/.zshrc /home/fernando/.zshrc
cp ~/.zshrc /home/codi/.zshrc
chown fernando:fernando /home/fernando/.zshrc
chown codi:codi /home/codi/.zshrc
```
