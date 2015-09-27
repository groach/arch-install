#Arch Linux MBP Retina Install
* Retina Early 2013
* Model A1398 (EMC 2673)
* Broadcom BCM4311 (14e4:4311)

##References
* [github.com/terlar](https://gist.github.com/terlar/6143325)
* [Arch Linux with rMBP](https://vec.io/posts/use-arch-linux-and-xmonad-on-macbook-pro-with-retina-display)
* [bsofdth/mac-arch](https://github.com/bsofdth/mac-arch)
* [Broadcom Wireless Driver](https://wiki.archlinux.org/index.php/Broadcom_wireless)

##Preparation

* [arch-linux-YYYY.MM.DD-dual.iso](https://www.archlinux.org/download/)
* [A Bootable Arch Flash Drive](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx)
* [rEFInd](http://www.rodsbooks.com/refind/getting.html)
* [Thunderbolt Ethernet Adapter](http://www.amazon.com/Thunderbolt-to-Gigabit-Ethernet-Adapter/dp/B008ALA6DW)

##Install and customize rEFInd 

Unzip and run install.sh . **Reboot**.

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

		menuentry "Arch Linux" {
		  icon /EFI/refind/rEFInd-minimal/icons/os_arch.png
		  loader vmlinuz-linux
		  initrd initramfs-linux.img
		  options "rw root=UUID=dfb2919d-ff78-48db-a8a7-23f7542c343a loglevel=3"
		}
		include refind-minimal/theme.conf

##Arch Install

###Useful commands
		lsblk -f #mounted disks and partitions
		lspci #wifi chipset info

###Partitions
		cgdisk /dev/sda

* Create `Boot`, `Root`, `Home` partitions. All `Linux Filesystem` type (8300) 
* `Write` and then `Quit` cgdisk

		lsblk -f #make note of partition ids!
		mkfs.ext4 /dev/sda5 #Boot
		mkfs.ext4 /dev/sda6 #Root
		mkfs.ext4 /dev/sda7 #Home
		
		mount /dev/sda6 /mnt
		mkdir /mnt/boot && mount /dev/sda5 /mnt/boot
		mkdir /mnt/home && mount /dev/sda7 /mnt/home

###Install

		pacstrap /mnt base base-devel
		genfstab -p /mnt >> /mnt/etc/fstab
		arch-chroot /mnt /bin/bash
		
		# computer name
		echo YOUR_COMPUTER_NAME > /etc/hostname
		
		# customize locales
		ln -s /usr/share/zoneinfo/America/New_York /etc/localtime
		vi /etc/locale.gen
		locale-gen
		echo LANG=en_US.UTF-8 > /etc/locale.conf
		export LANG=en_US.UTF-8
		
		mkinitcpio -p linux
		
		# rEFInd
		echo '"Arch Linux" "root=/dev/sdXY resume=/dev/sdXZ rw splash loglevel=3 rootflags=data=writeback libata.force=noncq acpi_osi=Linux acpi=force acpi_enforce_resources=lax i915.modeset=1 i915.enable_rc6=1 i915.enable_fbc=1 i915.lvds_downclock=1 initrd=boot/initramfs-linux.img"' > /boot/refind_linux.conf
		
		# users
		useradd -m -g users -G wheel -s /bin/bash username
		
		# passwords
		passwd
		passwd username
		

###Broadcom Drivers

* [Download Snapshot](https://aur.archlinux.org/packages/b43-firmware/) and copy to `~/builds`

		$ cd ~/builds
		$ tar -xvf b43-firmware.tar.gz
		$ makepkg -sri



		
