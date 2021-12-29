# Playing with the NUCLEO-F302R8 board from ST Microelectronics

STM32 board: NUCLEO-F302R8
Programmer: STLINK v2-1 (USB programmer attached to NUCLEO-F302R8)

Host: Raspberry Pi (Model B)
OS: Rasbian 10 (buster)
Kernel: 4.19

## Set up open source tools for STLINK USB programmer on Raspberry Pi [1]

stlink is an open source toolset to program and debug STM32 devices and boards.
It supports several STLINK programmer boards, which use a microcontroller chip to translate commands from USB to JTAG/SWD.
One of them is STLINK/V2-1 that is on the NUCLEO-F302R8 board.
The STLINK toolset includes:
- `st-info` - programmer and chip information tool
- `st-flash` - flash manipulation tool
- `st-trace` - logging tool to record information on execution
- `st-util` - GDB server
- `stlink-lib` - communication library
- `stlink-gui` - GUI interface _[optional]_

### 1. Common requirements

Install the following essential packages (for complete list refer to [1]):

- git
- gcc (C-compiler, already present)
- build-essential (on Debian based distros)
- cmake (3.4.2 or later, use the latest version)
- libusb-1.0 (already present)
- libusb-1.0-0-dev (development headers for building)

```
$ sudo apt install git gcc build-essential cmake libusb-1.0 libusb-1.0-0-dev
```

### 2. Download git repository

Change to your desired directory and invoke:


```
$ git clone https://github.com/stlink-org/stlink.git
```

### 3. Build and install the STLINK tools

Install the tools by invoking the following commands:

```
$ cd stlink         # change into the project source directory
$ make clean        # required by some linux distros
$ make release      # create the Release target
$ sudo make install # install the package with complete system integration
```

To remove:
```
$ sudo make uninstall # perform a clean uninstall of the package from the system
$ make clean          # remove the build folder
```

### 4. Set device access permissions and the role of udev

By default most distributions don't allow access to USB devices. The udev rules (that create device nodes) are necessary to run the tools without root permissions.
This is achieved by adding an user (who will try to access the STLINK USB programmer) to the `plugdev` group.

```
$ sudo usermod -aG plugdev <user>  # make <user> a member of the plugdev
 group
$ sudo groups <user>               # verify the membership of <user>
```

The udev rules are located in the subdirectory `config/udev/rules.d` and are automatically installed with `sudo make install` command on linux.
If not, then install and load these rules:

```
$ ls -la /etc/udev/rules.d/49-stlink*                  # check if udev rules are installed
$ sudo cp -a config/udev/rules.d/* /etc/udev/rules.d/  # deploy the udev rules
$ sudo udevadm control --reload-rules                  # reload and trigger them
$ sudo udevadm trigger
```

If you connect the USB programmer to one of the USB port of your Raspberry Pi computer, then udev will create device node files like `/dev/stlinkv3_XX`, `/dev/stlinkv2_XX`, `/dev/stlinkv1_XX` depending on STLINK USB device.

```
$ lsusb
Bus 001 Device 005: ID 0483:374b STMicroelectronics ST-LINK/V2.1
Bus 001 Device 004: ID 7392:7811 Edimax Technology Co., Ltd EW-7811Un 802.11n Wireless Adapter [Realtek RTL8188CUS]
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. SMSC9512/9514 Fast Ethernet Adapter
Bus 001 Device 002: ID 0424:9512 Standard Microsystems Corp. SMC9512/9514 USB Hub
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub

$ ll /dev/stlink*
lrwxrwxrwx 1 root root  3 Dec 29 10:42 /dev/stlinkv2-1_ -> sda
lrwxrwxrwx 1 root root 11 Dec 29 10:42 /dev/stlinkv2-1_0 -> bsg/0:0:0:0
lrwxrwxrwx 1 root root 15 Dec 29 10:42 /dev/stlinkv2-1_3 -> bus/usb/001/005

$ ll /dev/tty[UA]*
crw-rw-rw- 1 root dialout 166,  0 Dec 29 10:42 /dev/ttyACM0
crw--w---- 1 root tty     204, 64 Dec 29 10:33 /dev/ttyAMA0
```

## Simple tests

### 1. Probing the STM32F302 Nucleo board (with on-board STLINK programmer):

```
$ st-info --probe
---------- old ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x1ffff7cc
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

---------- new ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x0
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

Found 1 stlink programmers
  version:    V2J23S6
  serial:     066CFF524951775087233153
  flash:      16777216 (pagesize: 2048)
  sram:       40960
  chipid:     0x0439
---------- old ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x1ffff7cc
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

---------- new ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x0
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

  descr:      F301/F302/F318
```

### 2. Back up the actual flash content

A memory mapping of STM32F302x6/8 shows that the flash memory starts at address of 0x08000000 and has size of 65536 bytes (0x10000).
Refer to datasheet (page 48, DS9896 Rev 8) on the ST website [4].

```
$ st-flash read firmware.bin 0x08000000 0x10000

st-flash 1.7.0-126-gcb0f91a
---------- old ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x1ffff7cc
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

---------- new ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x0
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

2021-12-29T17:33:03 INFO common.c: F301/F302/F318: 40 KiB SRAM, 16384 KiB flash in at least 2 KiB pages.
2021-12-29T17:33:03 INFO common.c: read from address 0x08000000 size 65536
```

The content of the binary can be checked with `hexdump`:

```
$ hexdump -C myblink.bin | less

00000000  00 40 00 20 9d 13 00 08  ed 16 00 08 ef 16 00 08  |.@. ............|
00000010  f1 16 00 08 f3 16 00 08  f5 16 00 08 00 00 00 00  |................|
00000020  00 00 00 00 00 00 00 00  00 00 00 00 f7 16 00 08  |................|
00000030  f9 16 00 08 00 00 00 00  fb 16 00 08 fd 16 00 08  |................|
00000040  ed 13 00 08 ed 13 00 08  ed 13 00 08 ed 13 00 08  |................|
*
```

### 3. Deploy an example provided by STMicroelectronics

Download the 'STSW-STM32143' package, which is zipped as `en.stsw-stm32143_v1.2.1.zip`, from the official web site of ST Microelectronics (you need to be registered as an user). Choose `NUCLEO-F302R8` as target board.

Unzip it into the `en.stsw-stm32143_v1.2.1` directory.

Locate the `IO_Toggle.hex` file in the `en.stsw-stm32143_v1.2.1/STM32_Nucleo_FW_V1.2.1/Projects/NUCLEO-F302R8/IO_Toggle/Binary` directory and flash it to your STM32F302 Nucleo board.
This example demonstrates how to toggle IO with two different frequency by using Delay() function which is implemented based on the SysTick end-of-count event (`en.stsw-stm32143_v1.2.1/STM32_Nucleo_FW_V1.2.1/Projects/NUCLEO-F302R8/IO_Toggle/readme.txt`).

When starting program, LD2 remain toggling with a frequency equal to 10Hz (a delay of 50ms is put).
On press of the User button (B1) mounted on NUCLEO-F302R8 board, LD2 will toggle with a frequency equal to ~2.5Hz(a delay of 200ms is put). This is done in an infinite loop.

```
$ st-flash --format=ihex write IO_Toggle.hex

st-flash 1.7.0-126-gcb0f91a
---------- old ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x1ffff7cc
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

---------- new ------------
# Chip-ID file for F301/F302/F318
#
chip_id 0x439
description F301/F302/F318
flash_type  1
flash_size_reg 0x0
flash_pagesize 0x800
sram_size 0xa000
bootrom_base 0x1fffd800
bootrom_size 0x2000
option_base 0x1ffff800
option_size 0x10
flags 2

2021-12-29T18:37:56 INFO common.c: F301/F302/F318: 40 KiB SRAM, 12346 KiB flash in at least 2 KiB pages.
2021-12-29T18:37:56 INFO common.c: Attempting to write 2864 (0xb30) bytes to stm32 address: 134217728 (0x8000000)
2021-12-29T18:37:56 INFO common.c: Flash page at addr: 0x08000000 erased
2021-12-29T18:37:56 INFO common.c: Flash page at addr: 0x08000800 erased
2021-12-29T18:37:56 INFO common.c: Finished erasing 2 pages of 2048 (0x800) bytes
2021-12-29T18:37:56 INFO common.c: Starting Flash write for VL/F0/F3/F1_XL
2021-12-29T18:37:56 INFO flash_loader.c: Successfully loaded flash loader in sram
2021-12-29T18:37:56 INFO flash_loader.c: Clear DFSR
2021-12-29T18:37:56 INFO flash_loader.c: Clear CFSR
2021-12-29T18:37:56 INFO flash_loader.c: Clear HFSR
2021-12-29T18:37:56 INFO common.c: Go to Thumb mode
  2/  2 pages written
2021-12-29T18:37:56 INFO common.c: Starting verification of write complete
2021-12-29T18:37:56 INFO common.c: Flash written and verified! jolly good!
```

Once flashing is complete, press the on-board Reset button (B2).
You will see that the LD2 will be blinking fast!
Just toggle blink speed with the User button (B1).

## Troubleshooting

### 1. STLINK tools (st-*) return error because of missing `libstlink.so.1` [3]

If you have a following error when you are launching STLINK tools, then update your library cache by running `ldconfig`:

```
$ st-info --probe
st-info: error while loading shared libraries: libstlink.so.1: cannot open shared object file: No such file or directory

$ ldd /usr/local/bin/st-info
	/usr/lib/arm-linux-gnueabihf/libarmmem-${PLATFORM}.so => /usr/lib/arm-linux-gnueabihf/libarmmem-v6l.so (0xb6f83000)
	libstlink.so.1 => not found                                                # missing library location
	libusb-1.0.so.0 => /lib/arm-linux-gnueabihf/libusb-1.0.so.0 (0xb6f5e000)
	libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb6e10000)
	libudev.so.1 => /lib/arm-linux-gnueabihf/libudev.so.1 (0xb6de0000)
	libpthread.so.0 => /lib/arm-linux-gnueabihf/libpthread.so.0 (0xb6db6000)
	/lib/ld-linux-armhf.so.3 (0xb6f96000)
	librt.so.1 => /lib/arm-linux-gnueabihf/librt.so.1 (0xb6d9f000)

$ ll /usr/local/lib/libstlink.*
-rw-r--r-- 1 root root 169086 Dec 29 10:05 /usr/local/lib/libstlink.a
lrwxrwxrwx 1 root root     14 Dec 29 10:08 /usr/local/lib/libstlink.so -> libstlink.so.1
lrwxrwxrwx 1 root root     18 Dec 29 10:08 /usr/local/lib/libstlink.so.1 -> libstlink.so.1.7.0
-rw-r--r-- 1 root root 146040 Dec 29 10:06 /usr/local/lib/libstlink.so.1.7.0
```

The missing object file (library) was built and located in the `/usr/local/lib` directory, but it is still not in the search paths of `/etc/ld.so.cache`.
You must make sure that `/usr/local/lib` is added into the search paths and run `ldconfig` to update the library cache.

```
$ cat /etc/ld.so.conf
include /etc/ld.so.conf.d/*.conf

$ cat /etc/ld.so.conf.d/libc.conf
# libc default configuration
/usr/local/lib                        # location is already there


$ sudo ldconfig                       # update library cache

$ ldd /usr/local/bin/st-info
	/usr/lib/arm-linux-gnueabihf/libarmmem-${PLATFORM}.so => /usr/lib/arm-linux-gnueabihf/libarmmem-v6l.so (0xb6f38000)
	libstlink.so.1 => /usr/local/lib/libstlink.so.1 (0xb6f09000)
	libusb-1.0.so.0 => /lib/arm-linux-gnueabihf/libusb-1.0.so.0 (0xb6ee4000)
	libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb6d96000)
	libudev.so.1 => /lib/arm-linux-gnueabihf/libudev.so.1 (0xb6d66000)
	libpthread.so.0 => /lib/arm-linux-gnueabihf/libpthread.so.0 (0xb6d3c000)
	/lib/ld-linux-armhf.so.3 (0xb6f4b000)
	librt.so.1 => /lib/arm-linux-gnueabihf/librt.so.1 (0xb6d25000)
```

Links:

1. Git [repository](https://github.com/stlink-org/stlink) of the open source STLINK toolset
2. [Compilation instructions](https://github.com/stlink-org/stlink/blob/develop/doc/compiling.md) of the open source STLINK toolset
3. [Issue](https://github.com/stlink-org/stlink/issues/478) relevant to missing `libstlink.so.1`
4. STM32F302x6/8 [datasheet](https://www.st.com/resource/en/datasheet/stm32f302r6.pdf)
5. Another useful [link](https://cybergibbons.com/hardware-hacking/reading-and-writing-firmware-on-an-stm32-using-swd/) to flash STM32 boards
