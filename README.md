#Arch Linux MBP Retina Install
* Retina Early 2013
* Model A1398 (EMC 2673)
* Broadcom BCM43xx 1.0 (7.15.166.24.3)

##References
* [github.com/terlar](https://gist.github.com/terlar/6143325)
* [Arch Linux with rMBP](https://vec.io/posts/use-arch-linux-and-xmonad-on-macbook-pro-with-retina-display)
* [bsofdth/mac-arch](https://github.com/bsofdth/mac-arch)
* [Broadcom Wireless Driver](https://wiki.archlinux.org/index.php/Broadcom_wireless)

##Preparation

* [arch-linux-YYYY.MM.DD-dual.iso](https://www.archlinux.org/download/)
* [A Bootable Arch Flash Drive](http://www.ubuntu.com/download/desktop/create-a-usb-stick-on-mac-osx)
* [rEFInd](http://www.rodsbooks.com/refind/getting.html)

##Install and customize rEFInd 

Unzip and run install.sh . **Reboot**.

		sudo mount -t msdos /dev/disk0s1 /Volumes 

Edit `/Volumes/EFI/refind/refind.conf`:

		#uncomment/change the following lines
		scan_delay 1
		timeout 5
		
		
###[Minimalistic theme](https://github.com/EvanPurkhiser/rEFInd-minimal) Install

		cd /Volumes/EFI/refind
		git clone git@github.com:EvanPurkhiser/rEFInd-minimal.git

Add the following to the end of `refind.conf`

		menuentry "Arch Linux" {
		  icon /EFI/refind/rEFInd-minimal/icons/os_arch.png
		  loader vmlinuz-linux
		  initrd initramfs-linux.img
		  options "rw root=UUID=dfb2919d-ff78-48db-a8a7-23f7542c343a loglevel=3"
		}
		
Add minimal theme to `refind.conf`

		echo "include refind-minimal/theme.conf" >> refind.conf 


		
