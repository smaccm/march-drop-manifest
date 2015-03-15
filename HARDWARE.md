# CAmkES and seL4 on Odroid-XU Hardware

In order to run on hardware you will need an Odroid-XU from [hardkernel](http://www.hardkernel.com/).  You will need a appropriate serial to USB cable ([for example](http://www.hardkernel.com/main/products/prdt_info.php?g_code=G134111883934)) *and* a micro-USB cable to connect the Odroid-XU to your host computer.  

## Prerequisites

<a name="hostsoftware"></a>
### Host Software

#### Install `minicom`, `fastboot`, and `u-boot-tools`:

	sudo apt-get install minicom android-tools-fastboot u-boot-tools

#### Configure minicom:

Figure out what port you will connect your Odroid-XU's UART to.

  * typically this is `/dev/ttyUSB0`
  * if not, then plug in the UART cable to your Odroid-XU and the host computer
  * run `dmesg` and see what device name was assigned to it
  * use this value below for the `pu port` configuration option

Create a minicom configuration file:

	cat << EOF > ~/.minirc.odroid
	pu port             /dev/ttyUSB0
	pu baudrate         115200
	pu bits             8
	pu parity           N
	pu stopbits         1
	pu rtscts           No
	EOF

If you need to (re)configure from within minicom these settings are:

	115200 8N1, No HW/SW flow control

#### Configure fastboot

	echo SUBSYSTEM=="usb", ATTR{idVendor}=="18d1", MODE=="0666", GROUP=="users" | sudo tee /etc/udev/rules.d/40-odroidxu-fastboot.rules

## Connect Odroid-XU to Host

Connect a UART cable to the Odroid-XU's UART port (Serial console (debug) in the picture below) and to the host computer.

Connect a micro-USB cable to the Odroid-XU's USB port (USB 3.0 OTG in the picture below) and the host computer.

![Odroid-XU picture](http://www.cnx-software.com/wp-content/uploads/2013/08/ODROID-XU.jpg)

Start minicom on host in a separate window:

	minicom odroid

*Note:* if your user is not already in the `dialout` group you may need to run minicom with `sudo` or add your user to the `dialout` group (for example, by editing `/etc/group`).

### Flash u-boot on Odroid-XU

The Odroid-XU seL4 kernel requires an updated u-boot bootloader (to boot in HYP-mode) on the hardware.  You will have to reflash a new image before you run on the Odroid-XU.  You only have to do this once, unless you replace the u-boot again.

You will need updated bl2 and u-boot images. Contact us if you need to get these updated bl2 and u-boot images.

 * bl2: `odroid/smdk5410-spl.bin.signed`
 * u-boot: `odroid/u-boot.bin`

*Note:* You will need to install and configure `minicom` and `fastboot` on your host before doing this.  See [Prerequisites: Host Software](#hostsoftware).

The procedure to update u-boot is:

1. Connect a UART cable from the Odroid-XU to your host machine.  
2. Start `minicom odroid` in the host in a separate window
3. Connect a micro-USB cable from the Odroid-XU to your host machine
4. Boot the Odroid-XU into u-boot and enter the command `fastboot` in minicom
  * On the host machine, run `sudo fastboot devices` to check the
    connection. If it doesn't work, try restarting the Odroid-XU.
5. On the host, run:

		sudo fastboot flash bl2 smdk5410-spl.bin.signed
 		sudo fastboot flash bootloader u-boot.bin
	 	sudo fastboot reboot

6. When the Odroid-XU reboots, in minicom press Space or Enter to stop it from automatically booting into Linux.  You can type `fastboot` to start fastboot at this point instead.  However doing this everytime you reboot can get annoying, so reconfigure the boot command such that it automatically runs fastboot instead.  At the u-boot prompt do the following:

	setenv bootcmd fastboot
	saveenv

Note, if you do boot into Linux, some versions of Linux come with a boot.ini script that automatically replaces u-boot. Disabling this feature requires some simple
modification of the boot.ini file which lives on the SD/eMMC. This file can be modified with any text editor.


## Installing Linux root file-system image on SD (or eMMC)

### Get the Linux root file-system image

Get the image off dropbox (https://www.dropbox.com/s/hkduec0ezi7i2ux/smaccm_demo.img.gz)

### Install on SD

Insert the SD into the host machine. Note which device it is, for example `dev/sdb` (use `dmesg` to help). Then do:
       
       gunzip -c smaccm_demo.img.gz | dd of=<SD device file>; sync

Mount the first first partition of the SD card, and modify boot.ini to remove the line `run verify`.  This will make sure that if Linux boots directly it will not reinstall u-boot and overwrite our HYP enabled version.

Unmount and remove the SD card from the host machine.

Plug the SD card into the Odroid-XU.

#### Configure Linux in VM to load root file-system from SD

Note that by default Linux in the VM will try to use the SD card as the root file-system.  If you've changed this and need to set it back, then set the appropriate configuration option `CONFIG_VM_ROOTFS_MMCBLK1P2` using either `make menuconfig` or in the `.config` file or `vm_defconfig` file.

### Install on eMMC

Installing to eMMC is similar to SD. You use an eMMC to SD adaptor and plug that nto the host machine.

#### Configure Linux in VM to load root file-system from eMMC

Note that by default Linux in the VM will try to use the SD card as the root file-system.  To use the eMMC set the appropriate configuration option `CONFIG_VM_ROOTFS_MMCBLK0P2` using either `make menuconfig` or in the `.config` or `vm_defconfig` file.



