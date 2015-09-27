#Arch Linux MBP Retina Install
* 15" MBP Retina Early 2013
* Model A1398 (EMC 2673)
* Broadcom BCM4311 (14e4:4311)

##References
* [Official Arch Documentation](https://wiki.archlinux.org/index.php/Installation_guide)
* [terlar MBP Notes](https://gist.github.com/terlar/6143325)
* [Arch Linux with rMBP](https://vec.io/posts/use-arch-linux-and-xmonad-on-macbook-pro-with-retina-display)
* [bsofdth MBP Notes](https://github.com/bsofdth/mac-arch)
* [Broadcom Wireless Driver](https://wiki.archlinux.org/index.php/Broadcom_wireless)
* [Yaourt Install](https://www.digitalocean.com/community/tutorials/how-to-use-yaourt-to-easily-download-arch-linux-community-packages)

##Preparation

* [arch-linux-YYYY.MM.DD-dual.iso](https://www.archlinux.org/download/)
* [A Bootable Arch Flash Drive](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx)
* [rEFInd](http://www.rodsbooks.com/refind/getting.html)
* [Thunderbolt Ethernet Adapter](http://www.amazon.com/Thunderbolt-to-Gigabit-Ethernet-Adapter/dp/B008ALA6DW)


##Arch Install
Hookup thunderbolt ethernet and insert bootable arch flash drive. Reboot and hold `⌘`. Choose `EFI`. 

###Useful commands
		lsblk -f #mounted disks and partitions
		lspci #wifi chipset info

###Partitions
		cgdisk /dev/sda

**Create Partitions**

Create `Root`, `Home` partitions. All `Linux Filesystem` type (8300) 

`Write` and then `Quit` cgdisk

**Format and mount partitions**

		lsblk -f #make note of partition ids!
		
Create ext4 file systems on the `Root` and `Home` partitions and mount them:

		mkfs.ext4 /dev/sda5 #Root
		mkfs.ext4 /dev/sda6 #Home
		
		mount /dev/sda5 /mnt
		mkdir /mnt/home && mount /dev/sda6 /mnt/home

Mount existing EFI partition, found from `lsblk -f` as `Boot`:

		mkdir /mnt/boot && mount /dev/sdXN /mnt/boot

###Bootstrap

		pacstrap /mnt base base-devel
		genfstab -p /mnt >> /mnt/etc/fstab
		arch-chroot /mnt /bin/bash
		
		# computer name
		echo YOUR_COMPUTER_NAME > /etc/hostname
		
		# customize locales
		ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
		
		# uncomment needed locales
		vi /etc/locale.gen
		
		# generate locales & set language
		locale-gen
		echo LANG=en_US.UTF-8 > /etc/locale.conf
		export LANG=en_US.UTF-8
		
		# create new RAM disk
		mkinitcpio -p linux
		
		# set root password
		passwd

###Bootloader [rEFInd](https://wiki.archlinux.org/index.php/REFInd)

		pacman -S refind-efi
		refind-install

###yaourt Install (AUR)
Arch is very strict about which packages are made available via their standard package manager, `pacman`. Yaourt makes it easy to install community packages [(AUR)](https://wiki.archlinux.org/index.php/Arch_User_Repository#Installing_packages), without having to build them manually.

		sudo vi /etc/pacman.conf

Add to the bottom of file:

		[archlinuxfr]
		SigLevel = Never
		Server = http://repo.archlinux.fr/$arch

Save, quit. 

		sudo pacman -Sy yaourt

###Broadcom Drivers

		yaourt -S broadcom-wl-dkms

###Reboot

		exit #quit chroot environment
		unmount -R /mnt
		reboot

##Post Install
###Users
		useradd -m -G wheel -s /bin/bash groach #create user 
		passwd groach #set password

###[Intel Microcode Update](https://wiki.archlinux.org/index.php/Microcode)
		pacman -S intel-ucode
		vi /boot/refind_linux.conf
		
		##Example
		"Boot with standard options" "ro root=UUID=(...) quiet initrd=intel-ucode.img initrd=initramfs-linux.img"

###Wi-Fi
		sudo wifi-menu -o wlp3s0
		sudo systemctl enable netctl-auto@wlp3s0.service

###Basic Tools
		sudo pacman -S alsa-utils powertop dnsutils net-tools acpi openssh unzip unrar cronie git ack
		sudo systemctl enable sshd cronie

###Drivers
		yaourt -S acpid xf86-video-intel broadcom-wl-dkms xf86-input-mtrack-git macfanctld-git
		sudo cp /this/repo/xorg.conf.d/50-synaptics.conf /etc/X11/xorg.conf.d/
		sudo systemctl enable acpid macfanctld
		
		yaourt -S pacman -S bluez bluez-libs bluez-utils

##GUI
###Xorg
		sudo pacman -S xorg-server xorg-xrdb libnotify xbindkeys xorg-xmodmap
		sudo cp /this/repo/xorg.conf.d/10-monitor.conf /etc/X11/xorg.conf.d/

		
##Customize rEFInd 

Unzip and run install.sh,  **Reboot**.

###[Minimalistic theme](https://github.com/EvanPurkhiser/rEFInd-minimal) Install

		sudo mount -t msdos /dev/disk0s1 /Volumes 
		cd /Volumes/EFI/refind
		git clone git@github.com:EvanPurkhiser/rEFInd-minimal.git

Edit `/Volumes/EFI/refind/refind.conf`:

		#uncomment/change the following lines
		scan_delay 1
		timeout 5
		banner_scale fillscreen
		
Add the following to the end of `refind.conf`

		include refind-minimal/theme.conf
		



		
