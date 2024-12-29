# pcduino3B-kernel-instructions
An updated set of instructions to boot your pcduino3b with the most up to date kernel

# Make PcDuino3b Kernel:
 1. Create a directory to perform all your kernel making needs in, such as `mkdir pcduino3` and then enter that directory with `cd pcduino3`.
 2. Clone the repo of our ARM Kernel making script `git clone https://github.com/RobertCNelson/linux-dev.git`
 3. Modify `build_kernel.sh` `make_menuconfig ()` to the following:
```
make_menuconfig () {
	cd "${DIR}/KERNEL" || exit
	wget -c https://raw.githubusercontent.com/cglmrfreeman/pcduino3B-kernel-instructions/master/pcduino3_nano_config
	make ARCH=${KERNEL_ARCH} CROSS_COMPILE="${CC}" sunxi_defconfig
	make ARCH=${KERNEL_ARCH} CROSS_COMPILE="${CC}" menuconfig KCONFIG_CONFIG=pcduino3_nano_config
	if [ ! -f "${DIR}/.yakbuild" ] ; then
		cp -v .config "${DIR}/patches/defconfig"
	fi
	cd "${DIR}/" || exit
}
```
###### (This allows proper pcduino3b config settings to be applied to the kernel)
 4. `cd linux-dev` and run `./build_kernel.sh`  *(You may need to install lzop)*
 5. `cd ..` and clone the u-boot repo `git clone https://github.com/u-boot/u-boot.git` then enter the uboot directory `cd u-boot`.
 6. Cross compile u-boot *(you may need to install python38-devel, swig; change `(ver)` to whatever cross-compiler version was downloaded)*:
```
make CROSS_COMPILE="../linux-dev/dl/gcc-arm-(ver)-x86_64-arm-none-linux-gnueabihf/bin/arm-none-linux-gnueabihf-" Linksprite_pcDuino3_Nano_defconfig
make CROSS_COMPILE="../linux-dev/dl/gcc-arm-(ver)-x86_64-arm-none-linux-gnueabihf/bin/arm-none-linux-gnueabihf-" menuconfig
make CROSS_COMPILE="../linux-dev/dl/gcc-arm-(ver)-x86_64-arm-none-linux-gnueabihf/bin/arm-none-linux-gnueabihf-"
```
 7. Leave the `u-boot` folder with `cd ..` and format SD card with a program you are familair with (such as gparted).
 
|  | Type | Size |
|--|--|--|
| **Partition 1** | FAT16 | 100 MB |
| **Partition 2** | Ext4 | Remaining Space|

 8. Create a `boot.cmd` in the `pcduino3` folder with the following lines:
```
setenv bootargs console=ttyS0,115200 root=/dev/mmcblk0p2 rootwait panic=10 stdout=serial,vga stderr=serial,vga
load mmc 0:1 0x43000000 sun7i-a20-pcduino3-nano.dtb || load mmc 0:1 0x43000000 boot/sun7i-a20-pcduino3-nano.dtb
load mmc 0:1 0x42000000 zImage || load mmc 0:1 0x42000000 boot/zImage
bootz 0x42000000 - 0x43000000
```
9. Run `mkimage -C none -A arm -T script -d boot.cmd boot.scr` *(Install u-boot-tools if you need)*
10. Create "extlinux.conf"  in the `pcduino3` folder.
```
TIMEOUT 100
DEFAULT default
MENU TITLE Boot menu

LABEL default
	MENU LABEL Default
        LINUX /zImage
        FDT /sun7i-a20-pcduino3-nano.dtb
        APPEND root=/dev/mmcblk1p2

LABEL exit
	MENU LABEL Local boot script (boot.scr)
        LOCALBOOT 1
```
11. Extract sun7i-a20-pcduino3-nano.dtb from ""linux-dev/deploy/dbts.tar.gz" *(The names of the files may differ and this is easily done in a GUI rather than the command line)*
12. Write bootloader to SD card
`sudo dd if=./u-boot/u-boot-sunxi-with-spl.bin of=/dev/sdX bs=1024 seek=8` where `sdX` is your SD drive. Use program such as gparted to make sure of the proper letter to use here.
13.  Mount your partitions
```
sudo mkdir -p /media/boot/
sudo mkdir -p /media/rootfs/
sudo mount /dev/sdX1 /media/boot/
sudo mount /dev/sdX2 /media/rootfs/
```
Again where `sdX` is your SD drive.

14.  Copy boot stuff *(your filenames may differ)*:
```
sudo cp boot.scr /media/boot/
sudo cp extlinux.conf /media/boot/
sudo cp ./linux-dev/deploy/zImage /media/boot/
sudo cp ./dtb/location/sun7i-a20-pcduino3-nano.dtb /media/boot/
```
15.  Download fs from linaro. Latest (working) is here: 
>http://releases.linaro.org/debian/images/developer-armhf/17.06/linaro-stretch-developer-20170622-41.tar.gz
16. Extract the filesystem to rootfs with `sudo tar xpvf linaro-stretch-developer-20170622-41.tar.gz -C /media/rootfs/`
17. Extract the "modules.tar.gz" and put the "lib" folder in "/linux-dev/deploy" (Again most easily done from a GUI).
18. Put the lib modules in the rootfs lib with `sudo cp -R ./linux-dev/deploy/lib/ /media/rootfs/`
19. Edit fstab with `sudo nano /media/rootfs/etc/fstab` and add the following lines:
```
/dev/mmcblk0p2  /      auto  errors=remount-ro  0  1
/dev/mmcblk0p1  /boot  auto  errors=remount-ro  0  1
``` 
20. Now edit the network interfaces with `sudo nano /media/rootfs/etc/network/interfaces` and add
```
auto lo
iface lo inet loopback
 
auto eth0
iface eth0 inet dhcp
``` 
21.  Unmount your medias
```
sudo umount /media/boot/
sudo umount /media/rootfs/
```
22. **DONE!**

# Optional
## Install PiHole

1. After hooking up and ssh-ing into your pcduino3b, perform a `sudo apt-get update` and `sudo apt-get upgrade`.
2. After a `sudo reboot`,`sudo su` into root and perform the **One Step Automated Install:** `curl -sSL https://install.pi-hole.net | bash`
3. Follow the steps to install PiHole!
4. Enjoy!

>###### Old Linksprite instructions here (Archived in case Linksprite domain goes away) : https://archive.is/K8H1Z
>###### Minor instruction modifications and config file taken from here: https://github.com/nightseas/pcduino3_nano_a20
