# Guide-to-install-Arch-Linux-2024
This is a guide to install as well as dual boot Arch Linux with Windows. It can also work to install only Arch. You'll just have to skip some parts that i'll mention. For more updated and informated go to [Arch Linux Wiki](https://wiki.archlinux.org/title/Installation_guide)

## Some prerequisites
1. Disable Secure Boot. It can be enabled after installation.
2. Disable Fast Boot
3. Disable Hibernation.
4. If You are dual booting. Check if the EFI or the boot partition of windows is more than 100MB as it may cause problems. Refer to [Wiki](https://wiki.archlinux.org/title/Dual_boot_with_Windows) to increase partition size or see a video on youtube.

**DON'T TYPE THE DOLLAR SIGNS IT IS JUST FOR SHOWCASING THAT IT IS A COMMAND.**

## Step 1 - Pre-installation
### 1.1 Prepare an installation medium
1. Install the Latest Arch-Linux ISO file from [Arch ISO](https://archlinux.org/download/)
2. Now we'll use and Etcher tool to burn the iso file into a usb.
3. You can use [Balena Etcher](https://etcher.balena.io/), [Rufus](https://rufus.ie/en/) if you are on windows. If you are on Arch read the arch wiki to create an Arch USB [USB flash installation medium](https://wiki.archlinux.org/title/USB_flash_installation_medium)

### 1.2 - Boot the live environment
1. Plug the USB into a USB port and go into the BIOS by smashing that F2 or F12 or Del button (depends on the manufacturer) and change the boot priority so that the Arch USB is on number 1. Save and Exit
2. You'll see a grub menu with multiple things. Click Enter on the Arch linux installation medium (most probably the top one)
3. You'll see many thing just going saying ok. See if anything failed. After Some time you'll boot into the live environment.
4. You will be logged in on the first virtual console as the root user,

### 1.3 Set the console keyboard layout, font and basic stuff
1. The default Keymap is US. Available layouts can be listed with:
~~~
$ ls /usr/share/kbd/keymaps/**/*.map.gz
~~~
~~~
$ localectl list-keymaps
~~~
2. Set your Keyboard Layout. For example to set the German Layout. For more info [loadkeys](https://wiki.archlinux.org/title/Installation_guide#Set_the_console_keyboard_layout_and_font)
~~~
$ loadkeys de-latin1
~~~
3 . The Default font size is kinda small. So you might want to increase the size by 
~~~
$ setfont ter-132b
~~~
4. Connect to the internet,
  
   * Ensure your network interface is listed and enabled
      ~~~
       $ ip link
      ~~~
   * For wireless and WWAN, make sure the card is not blocked with [rfkill](https://wiki.archlinux.org/title/Network_configuration/Wireless#Rfkill_caveat).
   * Ethernet—plug in the cable.
   * Wi-Fi—authenticate to the wireless network using [iwctl](https://wiki.archlinux.org/title/Iwd#iwctl).
     ~~~
     $ iwctl
     [iwd]# device list
     [iwd]# station <device> scan
     [iwd]# station <device> get-networks
     [iwd]# station <device> connect <SSID>
     [iwd]# exit
     ~~~
     Replace device with the interface you wanna connect (For me it was wlan0) and SSID with the name of you network. after connecting it'll ask you the passphrase. It is the password of your WIFI.
   * Check if it is connected by pinging a website
     ~~~
     $ ping archlinux.org
     ~~~
5. In the live environment systemd-timesyncd is enabled by default and time will be synced automatically once a connection to the internet is established. Use timedatectl(1) to ensure the system clock is accurate:
~~~
$ timedatectl
~~~

### 1.4 Verify the boot mode
1. Type the following command to see is you are booted in UEFI mode. 
~~~
$ cat /sys/firmware/efi/fw_platform_size
~~~
If the command returns 64, then system is booted in UEFI mode and has a 64-bit x64 UEFI. If the command returns 32, then system is booted in UEFI mode and has a 32-bit IA32 UEFI; while this is supported, it will limit the boot loader choice to GRUB. If the file does not exist, the system may be booted in BIOS (or CSM) mode. Most probably for a normal user you'll want a 64-bit system so refer to the motherboard manual for this. If the file doesn't exist see if your computer supports UEFI mode by going into the BIOS. If it is enabled and it's still not showing recreate the USB by choosing a GPT partition type and UEFI enabled.

### 1.5 Partitioning the disks
1. Use lsblk to see your drives.It can be named sda,sdb,nvem0n1,nvme1n1. You got to know which drive you want to install arch on as we are going to wipe the drive afterwords so if you by mistake choose some other drive with important data on it, you are going to regret it. Just a **WARNING**.
~~~
$ lsblk
~~~
2.You can use fdisk command to get information about your drive partitions. It's really helpful while dual booting, I had a 1024MB EFI partition and 674MB windows recovery partition. you can see the partition info with it in order to differ EFI Partition that is FAT32 from recovery partition.
~~~
$ fdisk -l
~~~
3. Here I am going to make partition for boot (for the bootloader, this is not needed if you are **dualbooting** as there should be already a boot partition exists), swap (it'ss a partition that will work as RAM if we overload our RAM or if you want to use hibernation, you'll need it), root (it contains our system packages) and home (it will contain all the stuff like docs, downloads, pictures, etc in it). Some people dont like to make swap as most of the time its useless if you dont wanna hibernate as well as not making the home and using the root partition as home only. You can choose whatever you want but I am going to make all four partitions.
4. For my purpose im gonna choose nvmen0 as my drive to install which is 512GB nvme ssd. **Your drive may be different so choose very carefully**.
~~~
$ cfdisk /dev/nvmen0
~~~
5. It'll open a menu that will show your partition. To move we'll use the arrow keys. Now press ENTER on the unallocated space.
   * We are going to make the first partition 1024MiB. You dont need to make it if you are dualbooting. Press ENTER. Now with down arrow key move to next unallocated space.
   * We'll make the swap around 16GiB. It should be double of your RAM. For normal use I think 16GiB is enough even though you have more than 64 GIB of ram. For a proper swap size guide - [How Much Swap] (https://itsfoss.com/swap-size/)
   * Now go to type with arrow keys and select the linux swap.
   * We'll make the root partition 80GiB. Some people prefer 30GiB but for me it always seem less. You can choose whatever size you want just dont go under 30GiB.
   * We'll make the home partition and just press Enter without changing any values. After this i think i should have around 860GiB of home. You can trim it down, if you planning to install dual booting with other OSes.
   * **IMPORTANT** - Only the swap file should have linux swap written beside it. Other three should have **linux filesystem**.
   * Now click on write to write the partition table and  then quit.
   * We have Partitioned the drive.
   * Type lsblk again to see the partitioned drive.

### 1.6 Format the partitions
1. For this use case I am going to make the drives with [btrfs](https://docs.kernel.org/filesystems/btrfs.html) filesystem. Just understand btrfs give ability to take a snapshot so if we break our system we can roll back easily and I like btrfs. Snapshots are like backups of you system. Whenever you make changes to system, it takes a snapshot of previous state of the OS.
2. Lets start formatting
~~~
  $ lsblk
  $ mkfs.fat -F32 /dev/nvmen1p1                    // our boot partition. Dont need it if dual boot
  $ mkswap /dev/nvmen1p2                           // swap
  $ swapon /dev/nvmen1p2
  $ mkfs.btrfs /dev/nvmen1p3                       // Root
  $ mkfs.btrfs /dev/nvmen1p4                       // Home
~~~
3. Done with formatting the drive. type lsblk you'll see [SWAP] written beside the swap partition.

### 1,7 Mount the drive
~~~
$ mount /dev/nvmen1p3 /mnt                          // nvmen1p3 is the root partition
$ mount --mkdir /dev/nvmen1p1 /mnt/boot             // for linux only install (singleboot). Dont mount the Windows EFI partition now.         // nvmen1p1 is the boot partition
$ mount --mkdir /dev/nvmen1p4 /mnt/home             // nvmen1p4 is the home partition                                                         // Not required if you are not keeping home directory separate
~~~
Cool. Done with mounting the drives except the Windows EFI partition.

## Step 2 Installation
### 2.1 Ranking the mirrors and installing base packages
1. Archlinux uses servers to get the packages we need to install. Having good mirrors means packages install faster as well may prevent packages from out of date due to server being out of sync. It is generally not needed because the Archiso runs reflector when you connect to internet so you have a good copy of mirrorlist but it is generally better to resync the mirrors if you haven't done it in long time to remove any out of sync mirrors. [source](https://wiki.archlinux.org/title/Installation_guide#Select_the_mirrors)
2. There are two tools that we can use to rank mirrors and add them to our mirrorlist file. Reflector and Rankmirrors. Use Any one of them according to your use case.
3. Reflector - Retrieves the latest mirrorlist from the [MirrorStatus](https://archlinux.org/mirrors/status/) page, filters and sorts them by different parameters like speed, last sync, completion %, etc and overwrites /etc/pacman.d/mirrorlist. 
4. Rankmirrors - It fetches the list of the mirrors and ranks them locally based on many parameters. Reflector is good unless you are using private servers.
~~~
$ cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.backup   // Creates  backup of mirrorlist if something goes wronge
~~~
5. Using Rankmirrors - 
~~~
$ pacman -Sy                                                    // Linking with the mirrors
$ pacman -S pacman-contrib                                      // installing rankmirrors tools for ranking mirrors
$ rankmirrors -n 10 /etc/pacman.d/mirrorlist.backup > /etc/pacman.d/mirrorlist
~~~
6. Using Reflector - use reflector --help to see more options.
~~~
$ pacman -Sy                                               // linking with mirrors
$ pacman -S reflector rsync curl
$ sudo reflector --save /etc/pacman.d/mirrorlist --latest 10 --protocol https --sort rate  
~~~

7. Wait for some time. and we have ranked the mirrorlist.
8. If you got errors after updating the mirrorlist, you can revert back the changes with
~~~
$ cp /etc/pacman.d/mirrorlist.backup /etc/pacman/mirrorlist
~~~
9. Now installing the base packages
~~~
$ pacstrap -K /mnt base base-devel linux-zen linux-zen-headers linux-firmware sof-firmware intel-ucode nano vim git networkmanager dhcpcd bluez bluez-utils wpa_supplicant iwd network-manager-applet grub efibootmgr dosfstools mtools os-prober mesa gufw ufw tlp redshift man-db man-pages lvm2 neofetch
~~~
10. Go for a coffee break while it install if you have slow internet. Oh yeah btw replace "intel-ucode" with "amd-ucode" if you have an amd processor instead of intel. Also if you are installing on Desktop with no Wifi Support, remove "wpa_supplicant, iwd and bluez, bluez-utils". Bluez is for managing Bluetooth connections. Remove "tlp" as it is only for laptop battery optimizations. "sof-firmware" is for onboard audio mostly for laptop, but desktop will also work if you special audio controllers.
## Step 3 Configuring the System
### 3.1 FSTAB
1. Now we generate and **fstab** file that contains our UUID(house address) of our drives.
~~~
$ genfstab -U /mnt >> /mnt/etc/fstab
~~~
### 3.2 Chroot
Change root into the new system: 
~~~
$ arch-chroot /mnt
~~~
### 3.3 Timezone
~~~
$ ls /usr/share/zoneinfo/                                         // To show all Regions
$ ls /usr/share/zoneinfo/Asia/                                    // To show city in Asia
$ ln -sf /usr/share/zoneinfo/<Region>/<City> /etc/localtime       // Replace Region with Country and city with city
$ hwclock --systohc
~~~
### 3.4 Localization 
~~~
$ vim /etc/locale.gen                                   // Uncomment or delete the # before you language. For me it'll be en_US.UTF-8 UTF-8
$ locale-gen
$ echo LANG=en_US.UTF-8 > /etc/locale.conf
$ export LANG=en_US.UTF-8
~~~
### 3.5 Users and passwords
~~~
$ passwd                                // type the password it'll not show dont fear
$ useradd -m <username>                 // replace username with any name like fish
$ usermod -aG wheel,storage,power <username>
$ passwd <username>
$ visudo                                // EDITOR=nano visudo if you wanna use nano. 
~~~
1. Uncomment %wheel ALL=(ALL:ALL) ALL
2. Also add
~~~
Defaults timestamp_timeout=10             // just prompts user to type password again after a 10 minutes
~~~
3. Network configuration
~~~
$ echo <Hostname> > /etc/hostname                // replace <Hostname>
$ cat /etc/hostname
$ vim /etc/hosts
~~~
4. Add this in hosts file
~~~
127.0.0.1     localhost
::1           localhost
127.0.1.1     <Hostname>.localdomain    localhost
~~~
### 3.6 Mounting the boot directry and installing bootloader
~~~
$ mkdir /boot/efi
$ mount /dev/nvmen1p1 /boot/efi            // imagine if nvmen1p1 is the windows boot partition. its is not same as yours
~~~
1. I am going to use grub bootloader as systemd seems like a hassle. rEFInd seems cool to look but i wanna install with grub.
~~~
pacman -S grub efibootmgr dosfstools mtools os-prober      //ose-prober for dual booting. (not required to install as we done this in pacstrap command)
vim /etc/default/grub
~~~
2. Uncomment > GRUB_DISABLE_OS_PROBER=false
~~~
$ grub-install --target=x86_64-efi --bootloader-id=MyArchOS --recheck
$ grub-mkconfig -o /boot/grub/grub.cfg
~~~
## Step 4 - Finiishing up with live environment
### 4.1 Enabling things and unmounting
~~~
$ systemctl enable NetworkManager.service
$ exit
$ umount -lR /mnt        //unmount all the partition
~~~
### 4.2 reboot
~~~
$ reboot
~~~
Eject the USB drive and wait for grub bootloader to come. You'll see an Arch Linux entry by name **MyArchOS**. Press ENTER to enter Arch. Use your username as your login and the password you set. If you are using wireless connection you'll have to use **nmtui** to activate internet as iwctl is not present.
~~~
$ nmtui
~~~

# Done with the base installation. Refer to desktop directory to install DE/WM acc to your usage and installation of nvidia drivers would be in nvidia.md
