## BBB Debian Buster Environment Setup

### Explored Embedded HW Configuration:
https://github.com/ZoranStojsavljevic/MikroE_BeagleBone-Black-BSP_Integration/blob/master/BBB-debian_buster/overlay_examples/i2c2_sc16is740/MIKROE-3349/READM>

![](Images/beaglebone-mikrobus-cape.jpg)
![](Images/beaglebone-mikrobus-cape-SC16IS740.jpg)

MikroBus used is MikroBus 1 on MikroBus Cape4 BBB HW extension.

### Installing arm cross compiler on the Debian Buster host

To install on Debian Buster host, the following command is used:

	$ sudo apt-get install gcc-arm-linux-gnueabihf

### U-boot 04.2019 Overlays

The mandatory reading to understand U-boot 04.2019 overlays and environment
setup is posted here:

BeagleBone Black

https://www.digikey.com/eewiki/display/linuxonarm/BeagleBone+Black

Please, read this link very carefully!

#### Bootloader: U-Boot

Das U-Boot – the Universal Boot Loader: http://www.denx.de/wiki/U-Boot

eewiki.net patch archive: https://github.com/eewiki/u-boot-patches

Download U-Boot:

	user@localhost:~$
	git clone https://github.com/u-boot/u-boot
	cd u-boot/
	git checkout v2019.04 -b tmp

Or download U-Boot v2019.04 from:

	https://github.com/u-boot/u-boot/releases/tag/v2019.04

U-Boot Patches:

	user@localhost:~/u-boot$
	wget -c https://github.com/eewiki/u-boot-patches/raw/master/v2019.04/0001-am335x_evm-uEnv.txt-bootz-n-fixes.patch
	wget -c https://github.com/eewiki/u-boot-patches/raw/master/v2019.04/0002-U-Boot-BeagleBone-Cape-Manager.patch

	patch -p1 < 0001-am335x_evm-uEnv.txt-bootz-n-fixes.patch
	patch -p1 < 0002-U-Boot-BeagleBone-Cape-Manager.patch

Configure and Build:

	user@localhost:~/u-boot$
	make ARCH=arm CROSS_COMPILE=${CC} distclean
	make ARCH=arm CROSS_COMPILE=${CC} am335x_evm_defconfig
	make ARCH=arm CROSS_COMPILE=${CC}

	With ${CC} = arm-linux-gnueabihf- :

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 am335x_evm_defconfig
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 menuconfig
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8

	Optional for Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 am335x_evm_defconfig
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 menuconfig
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 menuconfig (if Linaro cross GCC compiler installed)

#### Optional Reading

U-Boot Overlays:

https://elinux.org/Beagleboard:BeagleBoneBlack_Debian#U-Boot_Overlays

BeagleBone uboot transplantation:

http://www.programmersought.com/article/19361176463/

### Setup microSD card

Erase partition table/labels on microSD card:

	$ sudo dd if=/dev/zero of=${DISK} bs=1M count=10

For the BBB board case, two files are generated: MLO and u-boot.img .

In order to flash the SD card!

MLO should reside at offset 1024KB (1MB) of the SD card. To place it there (assuming ${DISK} is /dev/sdX):

	Flash the MLO image into the SD card:
	$ sudo dd if=./u-boot/MLO of=${DISK} bs=128k count=1 seek=1; sync

	Flash the u-boot.img image into the SD card:
	$ sudo dd if=./u-boot/u-boot.img of=${DISK} bs=384k count=2 seek=1; sync

Similar:

Install Bootloader:

	$ sudo dd if=./u-boot/MLO of=${DISK} count=1 seek=1 bs=128k
	$ sudo dd if=./u-boot/u-boot.img of=${DISK} count=2 seek=1 bs=384k

For the Workshop:

	$ sudo dd if=MLO of=/dev/sdc count=1 seek=1 bs=128k
	$ sudo dd if=u-boot.img of=/dev/sdc count=2 seek=1 bs=384k

Create User / partition for the Debian Buster:

	$ echo "Create primary partition 1 for rootfs"
	$ sudo echo -e "n\np\n1\n\n+16384M\nw\n" | fdisk ${DISK}

	$ echo "Formatting primary partition ${DISK} for rootfs"
	$ sudo mkfs.ext4 -F ${DISK}

For the Workshop:

	$ echo "Create primary partition 1 for rootfs"
	$ sudo echo -e "n\np\n1\n\n+16384M\nw\n" | /sbin/fdisk /dev/sdc

	$ echo "Formatting primary partition sdb1 for rootfs"
	$ sudo mkfs.ext4 -F /dev/sdc1

### Supported Linux Kernels out of the box:

#### Pre-built kernels: (there are multiple options avaiable)

	$ cd /opt/scripts/tools/
	$ git pull
	$ sudo ./update_kernel.sh ## update to the latest kernel, supporting bb-overlays
	$ sudo reboot

#### Supported Linux Kernels: v4.19.x

v4.19.x-ti:

	# /opt/scripts/tools/update_kernel.sh --lts-4_19 --ti-channel

v4.19.x-ti + Real Time:

	# /opt/scripts/tools/update_kernel.sh --lts-4_19 --ti-rt-channel

v4.19.x mainline:

	# /opt/scripts/tools/update_kernel.sh --lts-4_19 --bone-channel

v4.19.x mainline + Real Time:

	# /opt/scripts/tools/update_kernel.sh --lts-4_19 --bone-rt-channel

#### Supported Linux Kernels: v5.4.24

	# /opt/scripts/tools/update_kernel.sh --lts-5-4.24

#### Tested Versions of dtc:

	v1.4.4
	v1.4.6
	v1.4.7
	v1.5.0
	v1.5.1

Known Broken: v1.4.5 (DO NOT USE)

### Building a Custom BB-kernel
https://github.com/RobertCNelson/bb-kernel

Current development is found under branches.

Example: https://github.com/RobertCNelson/bb-kernel/tree/am33x-v4.19

Execute the following to build the custom menuconfig:

	host$ git clone https://github.com/RobertCNelson/bb-kernel.git
	host$ cd bb-kernel
	host$ git remote show origin
	host$ git checkout am33x-v4.19
	host$ ./build_kernel.sh

Please, read very carefully the following net pointer:

https://github.com/ZoranStojsavljevic/BBB_Workshop_Examples/blob/master/Kernel_Porting_Guide/Kernel_Porting_Guide.md

### Common BeagleBone Black Linux kernel requirements

Here are the custom changes for menuconfig, to be done in order to have proper kernel for the example:

https://github.com/ZoranStojsavljevic/MikroE_BeagleBone-Black-BSP_Integration/blob/master/BBB-debian_buster/overlay_examples/KERNEL.md

Please, do note that the customized kernel must be built in order to achieve the desired
functionality for the most MikroE CLICK boards:

	[1] The standard defconfig .config  will change in order to include CLICK silicon
	    driver options
	[2] In most the cases the Device Tree Source (DTS) device fragment either will be:
		[A] Included with the standard  .../arch/arm/boot/dts/am335x-boneblack.dts
		[B] Needs to be imported to the .../arch/arm/boot/dts/am335x-boneblack.dts
		[C] Needs to be included as DTS overlay

These requirements are calling for the customized kernel, which does NOT come out of the box!

#### Enabling OF_OVERLAY DTS overlay .config option

In the most common case used (to added kernel device driver having its own DTS representation),
DTS overlay is used. The kernel DTS overlay representation is the following:

	  │ Symbol: OF_OVERLAY [=y]
	  │ Type  : bool
	  │ Prompt: Device Tree overlays
	  │   Location:
	  │     -> Device Drivers
	  │ (2)   -> Device Tree and Open Firmware support (OF [=y])
	  │   Defined at drivers/of/Kconfig:92
	  │   Depends on: OF [=y]

They will appear as the following CONFIG options in the .config :

	CONFIG_OF_OVERLAY=y
	# CONFIG_OVERLAY_FS is not set

#### Disabling SERIAL_DEV_CRTL_TTYPORT .config option
https://cateee.net/lkddb/web-lkddb/SERIAL_DEV_CTRL_TTYPORT.html

Just disabling CONFIG_SERIAL_DEV_CTRL_TTYPORT gives a working /dev/ttySC0

	  │ Symbol: SERIAL_DEV_CTRL_TTYPORT [=n]
	  │ Type  : bool
	  │ Prompt: Serial device TTY port controller
	  │   Location:
	  │     -> Device Drivers
	  │       -> Character devices
	  │ (1)     -> Serial device bus (SERIAL_DEV_BUS [=y])
	  │   Defined at drivers/tty/serdev/Kconfig:14
	  │   Depends on: TTY [=y] && SERIAL_DEV_BUS [=y]=y

This (appears to be generic TTY driver) option is under investigation.

### Adding SC16IS7xx driver to the .config - Adding kernel .config fragments

These requirements call for the customized kernel, which does NOT come out of the box!

The DTS method used in this example is DTS overlay.

Please, execute the following command in the root tree source of the currently used BBB Linux kernel:

Change kernel .config (original defconfig used to build .config is omap2plus_defconfig):

	Debian:
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 menuconfig

	Optional for Fedora:
	ARCH=arm CROSS_COMPILE=arm-linux-gnu- make -j8 menuconfig
	ARCH=arm CROSS_COMPILE=arm-linux-gnueabihf- make -j8 menuconfig (if Linaro cross GCC compiler installed)

Please, find the following options and add them to be included in the kernel:

	  │ Symbol: SERIAL_SC16IS7XX [=y]
	  │ Type  : tristate
	  │ Prompt: SC16IS7xx serial support
	  │   Location:
	  │     -> Device Drivers
	  │       -> Character devices
	  │ (1)     -> Serial drivers
	  │   Defined at drivers/tty/serial/Kconfig:1083
	  │   Depends on: TTY [=y] && HAS_IOMEM [=y] && (SPI_MASTER [=y] && !I2C [=y] || I2C [=y])
	  │   Selects: SERIAL_CORE [=y]
	  │
	  │
	  │ Symbol: SERIAL_SC16IS7XX_CORE [=y]
	  │ Type  : tristate
	  │   Defined at drivers/tty/serial/Kconfig:1080
	  │   Depends on: TTY [=y] && HAS_IOMEM [=y]
	  │   Selected by [y]:
	  │   - SERIAL_SC16IS7XX_I2C [=y] && TTY [=y] && HAS_IOMEM [=y] && I2C [=y] && SERIAL_SC16IS7XX [=y]
	  │   - SERIAL_SC16IS7XX_SPI [=y] && TTY [=y] && HAS_IOMEM [=y] && SPI_MASTER [=y] && SERIAL_SC16IS7XX [=y]
	  │
	  │
	  │ Symbol: SERIAL_SC16IS7XX_I2C [=y]
	  │ Type  : bool
	  │ Prompt: SC16IS7xx for I2C interface
	  │   Location:
	  │     -> Device Drivers

They will appear as the following CONFIG options in the .config :

	CONFIG_SERIAL_SC16IS7XX_CORE=y
	CONFIG_SERIAL_SC16IS7XX=y
	CONFIG_SERIAL_SC16IS7XX_I2C=y
	CONFIG_SERIAL_SC16IS7XX_SPI=y

### U-Boot Generic uEnv.txt file used (example overlays commented out by default)

https://github.com/ZoranStojsavljevic/MikroE_BeagleBone-Black-BSP_Integration/blob/master/BBB-debian_buster/overlay_examples/uEnv.txt

### Debian Root File System

	Debian Buster user: debian, passwd: temppwd
	Debian Buster root: root, passwd: root

Download Debian Root File System:

	$ wget -c https://rcn-ee.com/rootfs/eewiki/minfs/debian-10.4-minimal-armhf-2020-05-10.tar.xz
	$ tar -xvf debian-10.4-minimal-armhf-2020-05-10.tar.xz

### Copy the Root File System

Mounting point for the SDcard could be various paths:

	/media/rootfs/
	/run/media/vuser/rootfs/

Assuming /run/media/vuser/rootfs mounting point, the following should be done:

	$ sudo tar xfvp ./*-*-*-armhf-*/armhf-rootfs-*.tar -C /run/media/vuser/rootfs
	$ sync
	$ sudo chown root:root /run/media/vuser/rootfs
	$ sudo chmod 755 /run/media/vuser/rootfs/

WARNING: Please, do note that in the /run/media/vuser/rootfs/boot/ there is NO
any data, since the given Debian Root File System tends to be GENERIC Root
Tree, applicable for ANY ARM architecture!

### BBB P9 header I2C2 Overlay (BB-I2C2-SC16IS740-00A0.dts)
```
$ cat BB-I2C2-SC16IS740-00A0.dts

/*
 * Virtual cape for I2C2 on connector pins P9.12 P9.15 P9.19 P9.20
 *
 * This program is free software; you can redistribute it and/or modify
 * it under the terms of the GNU General Public License version 2 as
 * published by the Free Software Foundation.
 */

/dts-v1/;
/plugin/;

/ {
	compatible = "ti,beaglebone", "ti,beaglebone-black";

	// identification
	part-number = "BB-I2C2-SC16IS740";
	version = "00A0";

	// resources this cape uses
	exclusive-use =
		"P9.12",	// RESET
		"P9_15",	// INT
		"P9.19",	// i2c2_sda
		"P9.20",	// i2c2_scl
		"i2c2";		// hardware ip used

	/*
	 * Helper to show loaded overlays under: /proc/device-tree/chosen/overlays/
	 */
	fragment@0 {
		target-path="/";
		__overlay__ {

			chosen {
				overlays {
					BB-I2C2-SC16IS740-00A0 = "Fri May 19 19:35:36 2020";
				};
			};
		};
	};

	/*
	 * Free up the pins used by the cape from the pinmux helpers.
	 */
	fragment@1 {
		target = <&ocp>;
		__overlay__ {
			P9_12_pinmux { status = "disabled"; };	/* RESET */
			P9_15_pinmux { status = "disabled"; };	/* INT */
			P9_19_pinmux { status = "disabled"; };	/* i2c2_scl */
			P9_20_pinmux { status = "disabled"; };	/* i2c2_sda */
		};
	};

	fragment@2 {
		target = <&am33xx_pinmux>;
		__overlay__ {
			bb_gpio_pins: pinmux_bb_gpio_pins {
				pinctrl-single,pins = <
					0x078 0x2f	/* (PIN_INPUT | MUX_MODE7) -> P9_12 */
					0x040 0x2f	/* (PIN_INPUT | MUX_MODE7) -> P9_15 */
				>;
			};
			bb_i2c2_pins: pinmux_bb_i2c2_pins {
				pinctrl-single,pins = <
					0x17c 0x73	/* i2c2_sda, SLEWCTRL_SLOW | INPUT_PULLUP | MODE3 */
					0x178 0x73	/* i2c2_scl, SLEWCTRL_SLOW | INPUT_PULLUP | MODE3 */
				>;
			};
		};
	};

	fragment@3 {
		target = <&i2c2>;
		__overlay__ {
			pinctrl-names = "default";
			pinctrl-0 = <&bb_i2c2_pins>;

			/* this is the configuration part */
			// clock-frequency = <400000>;
			status = "disabled";

			#address-cells = <1>;
			#size-cells = <0>;

			/* add any i2c devices on the bus here */
		};
	};

	fragment@4 {
		target = <&i2c2>;
		__overlay__ {
			#address-cells = <1>;
			#size-cells = <0>;
			status = "okay";

			sc16is740: sc16is740@49 {
				compatible = "nxp,sc16is740";
				pinctrl-names = "default";
				pinctrl-0 = <&bb_gpio_pins>;
				reg = <0x49>; /* address */
				clocks = <&sc16is740_clk>;
				interrupt-parent = <&gpio1>;
				interrupts = <16 2>; /* IRQ_TYPE_FALLING_EDGE */
				#gpio-cells = <2>;
				clock-frequency = <400000>;

				sc16is740_clk: sc16is740_0_clk {
					compatible = "fixed-clock";
					#clock-cells = <0>;
					clock-frequency = <1843200>;
				};
			};
		};
	};
};
```

After importing this overlay, the following is an outcome on the target BBB platform:

	# i2cdetect -r 2

	WARNING! This program can confuse your I2C bus, cause data loss and worse!
	I will probe file /dev/i2c-2 using receive byte commands.
	I will probe address range 0x03-0x77.
	Continue? [Y/n] y
	     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
	00:          -- -- -- -- -- -- -- -- -- -- -- -- -- 
	10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	40: -- -- -- -- -- -- -- -- -- UU -- -- -- -- -- -- 
	50: 50 51 52 53 UU UU UU UU -- -- -- -- -- -- -- -- 
	60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- 
	70: -- -- -- -- -- -- -- -- 

#### SC16IS7xx Driver Problems
https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/log/drivers/tty/serial/sc16is7xx.c?h=v5.7-rc5

The problem in details is described here:

https://git.kernel.org/pub/scm/linux/kernel/git/next/linux-next.git/commit/drivers/tty/serial/sc16is7xx.c?h=next-20200515&id=4a37c0fcf5d403e7d9f5309ea970ea3c9074b5d8

```
author	Gustavo A. R. Silva <gustavo@embeddedor.com>	2020-02-12 18:46:11 -0600
committer	Greg Kroah-Hartman <gregkh@linuxfoundation.org>	2020-02-13 12:00:23 -0800
commit	4a37c0fcf5d403e7d9f5309ea970ea3c9074b5d8 (patch)
tree	36ec2b06790facbd2991ffc6423ea8aa463a792c /drivers/tty/serial/sc16is7xx.c
parent	2f202d03a5785ffaf894f9503193a3767ff88d88 (diff)
download	linux-next-4a37c0fcf5d403e7d9f5309ea970ea3c9074b5d8.tar.gz
serial: sc16is7xx: Replace zero-length array with flexible-array member
The current codebase makes use of the zero-length array language
extension to the C90 standard, but the preferred mechanism to declare
variable-length types such as these ones is a flexible array member[1][2],
introduced in C99:

struct foo {
        int stuff;
        struct boo array[];
};

By making use of the mechanism above, we will get a compiler warning
in case the flexible array does not occur last in the structure, which
will help us prevent some kind of undefined behavior bugs from being
inadvertently introduced[3] to the codebase from now on.

Also, notice that, dynamic memory allocations won't be affected by
this change:

"Flexible array members have incomplete type, and so the sizeof operator
may not be applied. As a quirk of the original implementation of
zero-length arrays, sizeof evaluates to zero."[1]

This issue was found with the help of Coccinelle.

[1] https://gcc.gnu.org/onlinedocs/gcc/Zero-Length.html
[2] https://github.com/KSPP/linux/issues/21
[3] commit 76497732932f ("cxgb3/l2t: Fix undefined behaviour")

Signed-off-by: Gustavo A. R. Silva <gustavo@embeddedor.com>
Link: https://lore.kernel.org/r/20200213004611.GA8748@embeddedor.com
Signed-off-by: Greg Kroah-Hartman <gregkh@linuxfoundation.org>
Diffstat (limited to 'drivers/tty/serial/sc16is7xx.c')
-rw-r--r--	drivers/tty/serial/sc16is7xx.c	2

1 files changed, 1 insertions, 1 deletions
diff --git a/drivers/tty/serial/sc16is7xx.c b/drivers/tty/serial/sc16is7xx.c
index 7d3ae31cc720..06e8071d5601 100644
--- a/drivers/tty/serial/sc16is7xx.c
+++ b/drivers/tty/serial/sc16is7xx.c
@@ -329,7 +329,7 @@ struct sc16is7xx_port {
	struct task_struct		*kworker_task;
	struct kthread_work		irq_work;
	struct mutex			efr_lock;
-	struct sc16is7xx_one		p[0];
+	struct sc16is7xx_one		p[];
 };

static unsigned long sc16is7xx_lines;
```
#### SERIAL_DEV_CRTL_TTYPORT config option
https://cateee.net/lkddb/web-lkddb/SERIAL_DEV_CTRL_TTYPORT.html

Just disabling CONFIG_SERIAL_DEV_CTRL_TTYPORT gives a working /dev/ttySC0 :

	  │ Symbol: SERIAL_DEV_CTRL_TTYPORT [=n]
	  │ Type  : bool
	  │ Prompt: Serial device TTY port controller
	  │   Location:
	  │     -> Device Drivers
	  │       -> Character devices
	  │ (1)     -> Serial device bus (SERIAL_DEV_BUS [=y])
	  │   Defined at drivers/tty/serdev/Kconfig:14
	  │   Depends on: TTY [=y] && SERIAL_DEV_BUS [=y]=y

The description of the CONFIG_SERIAL_DEV_CTRL_TTYPORT flag is given here:

https://cateee.net/lkddb/web-lkddb/SERIAL_DEV_CTRL_TTYPORT.html

#### Final Results

	root@arm:/home/debian# ls -al /proc/device-tree/chosen/overlays/
	total 0
	drwxr-xr-x 2 root root  0 May 19 20:07 .
	drwxr-xr-x 3 root root  0 May 19 20:07 ..
	-r--r--r-- 1 root root 25 May 19 20:07 BB-I2C2-SC16IS740-00A0
	-r--r--r-- 1 root root  9 May 19 20:07 name
	root@arm:/home/debian# ls -al /dev | grep ttySC0
	crw-rw----  1 root dialout 245,   0 May 19 20:06 ttySC0
	root@arm:/home/debian# uname -a
	Linux arm 5.4.24+ #2 PREEMPT Mon May 18 10:15:36 CEST 2020 armv7l GNU/Linux
	root@arm:/home/debian# dmesg | grep i2c
	[    1.556519] omap_i2c 4802a000.i2c: bus 1 rev0.11 at 100 kHz
	[    1.833350] i2c /dev entries driver
	[    2.227634] input: tps65217_pwr_but as /devices/platform/ocp/44c00000.interconnect/44c00000.interconnect:segment@200000/44e0b000.target-module/44e0b000.i2c/i2c-0/0-0024/tps65217-pwrbutton/input/input0
	[    2.229011] omap_i2c 44e0b000.i2c: bus 0 rev0.11 at 400 kHz
	[    2.263240] omap_i2c 4819c000.i2c: bus 2 rev0.11 at 100 kHz
	root@arm:/home/debian#
