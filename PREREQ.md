<!-- @LICENSE(NICTA_CORE) -->

# Prerequisites

Before building and running CAmkES-based systems you must have several prerequisite s oftware packages installed on the machine you will be using to build.  

## Linux

First of all we assume that you are running Linux.  We typically use some flavour of Ubuntu or other Debian-based system.  You could try using other Unix systems (including Mac OS) but your mileage may vary.  The following instructions assume Ubuntu.

### Docker

If you use Docker we have support for using a Docker image to build seL4 and CAmkES that is preconfigured with all the tools needed to download and build the code.  See [github.com/ikuz/docker-sel4-camkes](https://github.com/ikuz/docker-sel4-camkes).  

Note, however, the Docker image is not set up to connect to the Odroid-XU (see [`HARDWARE.md`](https://github.com/smaccm/march-drop-manifest/blob/master/HARDWARE.md) for general details of how to do this).

## Toolchains

Downloading CAmkES requires `git` and `repo`.

Building CAmkES reguires: an appropriate cross compiler, Python, Python packages (Jinja2, PLY, pyelftools), Haskell, Cabal (a Haskell package manager) and Haskell packages (MissingH, Data.Ordlist, Split).

### Install git and repo

    	sudo apt-get install git phablet-tools; 

Tell git who you are (replace `<email>` with your own email address):

     	git config --global user.email "<email>"    	

### Install a GCC cross-compiler

Do the following to install an appropriate development environment and GCC cross-compiler on Ubuntu 14.04:

   	sudo apt-get install build-essential gcc-multilib ccache ncurses-dev

	sudo apt-get install python-software-properties
	sudo add-apt-repository universe
	sudo apt-get update
	sudo apt-get install gcc-arm-linux-gnueabi
	sudo apt-get install gcc-arm-none-eabi

In order to support 32-bit binaries on 64-bit Linux do:

	sudo apt-get install lib32z1 lib32ncurses5 lib32bz2-1.0

### Install Qemu emulator

	sudo apt-get install qemu-system-arm qemu-system-x86

### Install Python and packages

	sudo apt-get install python python-pip python-tempita python-jinja2 python-ply
	sudo pip install --upgrade pip
	sudo pip install pyelftools

### Install Haskell and packages

	sudo apt-get install cabal-install ghc libghc-missingh-dev libghc-split-dev
	cabal update
	cabal install data-ordlist

### Install miscellaneous other requirements
	
	sudo apt-get install realpath libxml2-utils 

### Install minicom, fastboot, and u-boot-tools for accessing and loading on hardware:

	sudo apt-get install minicom android-tools-fastboot u-boot-tools

