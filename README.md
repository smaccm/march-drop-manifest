# SMACCM Red Team drop March 15 2015

# Quadcopter Mission Board

## Drop Contents

The Quadcopter Mission Board drop contains the following scenario:
 * seL4 running on the Odroid-XU (daughterboard optional)
 * an instance of virtualised Linux running. Linux has access to all USB devices and the eMMC/SD storage.  Its console output is redirected to the seL4 debug serial port.  Linux can be logged into through the console, or over the network (using either the Odroid-XU's Ethernet or a Wifi adapter plugged into a USB port). 
 * a set of CAmkES/seL4 components implementing a simple ping-pong scenario.  These components are generated from an AADL system description and user-provided C code.
 * native seL4 device drivers.
   * USB: optionally used by the Linux VM
   * PWM, timer: used by the ping-pong example
 * the ping-pong components run isolated from the Linux VM
 * Depending on a configuration option (`CONFIG_VM_USB`) the VM's USB access is either passthrough or goes through the native seL4 USB driver. By default passthrough is used for USB.
 * The eMMC/SD access always uses passthrough.  
 * When any of USB or eMMC are accessed using passthrough then Linux has access tot he DAM controller, which opens up a big security hole. The native USB driver helps to close this hole by avoiding USB passthrough, and a native eMMC controller will be developed to close this hole in the future.

## Prerequisites

See `PREREQ.md` for a detailed list of prerequisites required to download and build the code.

## Downloading and Building

Get the source code using:

    mkdir march-drop-manifest
    cd march-drop-manifest
    repo init -u https://github.com/smaccm/march-drop-manifest.git
    repo sync

Load the appropriate configuration

     make vm_defconfig
     make silentoldconfig

Build

     make

## Preparing Odroid-XU

See `HARDWARE.md` for detailed instructions on how to prepare the Odroid-XU to run this code.

## Loading and Running on Odroid-XU

Convert to an Odroid image

	cd images/
	mkimage -a 0x48000000 -e 0x48000000 -C none -A arm -T kernel -O qnx -d capdl-loader-experimental-image-arm-exynos5 odroid-image

Turn on Odroid-XU

Watch the Odroid-XU start in `minicom` window.  If necessary execute `fastboot` on the Odroid-XU by typing into `minicom`:

	fastboot

Execute `fastboot` on the host:

	sudo fastboot boot odroid-image

The system will run. It will boot up Linux, eventually giving a login prompt. You can login to Linux using the following credentials:

    login: odroid
    password: odroid

    login: root
    password: odroid

Note that the ping-pong components also write to the console, meaning that using the console to interact with Linux is awkward.  
It is easier to log into Linux over the network.

## Logging in over the network

### Ethernet connection to a network with DHCP

Connect the Odroid-XU using an Ethernet cable to a switch or network port, it will get an address using DHCP.  
Find out the address it was given and `ssh` to the Odroid-XU, then login.

### Ethernet to computer

Connect the Odroid-XU directly to another computer using an Ethernet cable.

Configure the network address on the computer:

	  sudo ifconfig eth0 192.168.0.30

Configure the network address on the Odroid-XU (via the console):

	  nmcli dev disconnect iface eth0;
	  sudo ifconfig eth0 192.168.0.10

`ssh` to the Odroid-XU and login:

      	ssh 192.168.0.10

### Wifi connecting to an Access Point

Plug a compatible Wifi adapter (see `HARDWARE.md`) into a USB port on the Odroid-XU.  Configure Wifi (via the console) ...

### Wifi in AdHoc mode

Plug a compatible Wifi adapter (see `HARDWARE.md`) into a USB port on the Odroid-XU.  Configure Wifi (via the console) ...

## Camera

The Linux image has drivers and an application for using the [Pixy camera | http://charmedlabs.com/default/pixy-cmucam5/]. 
Before using this, set up a network connection as above.

Then on the Odroid-XU do:

     /root/camera_demo/demo

On another machine (with Java installed (`sudo apt-get install java`), and able to connecto the Odroid-XU over the network):

   	git clone https://github.com/smaccm/camera_demo.git
	cd camera_demo
	java -jar SmaccmViewer.jar 192.168.0.10
   
Thi application opens a window showing the camera stream.

## Code overview

The Linux VM and ping-pong code can be found in `apps/vm`.  The CAmkES description is in `vm.camkes`, the component code is in `components/`.

Libraries used by the system are in `libs/`, seL4 is in `kernel/`, and CAmkES is in `tools/camkes`.

The original ping-pong application can be found in `apps/ping_pong`.  Besides the CAmkES code, this also contains the AADL files and original user code that the CAmkES code was generated from.

The Linux kernel used for booting is in `apps/vm/linux/linux`.
Device trees for this Linux are in `apps/vm/linux/linux-secure-dtb` `apps/vm/linux/linux-secure-vusb-dtb`. 
The former provides passthrough access to USB hardware, while the second paravirtualises USB access and uses the native seL4 USB driver.
Which one is used is determined by the `CONFIG_VM_USB` configuration paramter, which can be set using `make menuconfig` or in the `.config` or `vm_defconfig` file.  
By default, the `linux-secure-dtb` (passthrough USB) is used.

The Linux file-system image used can be found at: (https://www.dropbox.com/s/hkduec0ezi7i2ux/smaccm_demo.img.gz)