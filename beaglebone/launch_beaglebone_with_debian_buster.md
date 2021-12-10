# Launch BeagleBone Black with Debian Buster (10)

Requires:
- 4GB/8GB uSD card (FAT formatted)
- (optional) TTL-to-RS232 adapter cable

This instruction presents how to launch the BeagleBone Black board with Debian Buster (10) distribution. The same procedure can apply for other suitable images.

## Back up the content of the eMMC device [1]

**Note:** for troubleshooting visit the web site and read notices.

1. Get a uSD card.
2. Make sure that the partition of the uSD card is marked as active.
3. Download this ZIP file and extract it onto the uSD card.

```
$ wget https://s3.amazonaws.com/beagle/beagleboneblack-save-emmc.zip
$ unzip beagleboneblack-save-emmc.zip -d /media/$USER/save_emmc       # /media/$USER/save_emmc is mountpoint of the uSD card
```

4. Insert the uSD card into powered-off BeagleBone Black.
5. Hold down the 'S2' button and apply power to the board. Release the button after boot (user LEDs will blink).
6. The backup progress is signalled with user LEDs: USR0 blinks first in "heartbeat" (around several seconds), then blinks steadily (around 10 minutes), and finally stay ON.
Also if you connected serially with the board (refer to **How-to: Connect the BeagleBone Black to a host serially**), then you can see the progress in terminal.
7. Now the uSD can be removed. The eMMC content should be saved in the 'BBB-eMMC-XXXXX.img.gz' file. Store it in your backup location.

## Prepare uSD card with new image [2]

1. Download uSD image 'AM3358 Debian 10.3 2020-04-06 4GB SD IoT'

```
$ wget https://debian.beagleboard.org/images/bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz
```

2. Verify checksum

```
$ sha256sum bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz
22448ba28d0d58e25e875aac3b4e91eaef82e2d11c9d2c43d948ed60708f7434  bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz
```

3. Uncompress the image and write to the uSD card

```
$ unxz bone-debian-10.3-iot-armhf-2020-04-06-4gb.img.xz
$ sudo dd if=bone-debian-10.3-iot-armhf-2020-04-06-4gb.img of=/dev/mmcblk0 status=progress
```

Now you can boot from the uSD card. Refer to **How-to: Boot from the uSD card**.

## Flash the eMMC device with the uSD card image

1. After booting the uSD edit the '/boot/uEnv.txt' file so that a '#cmdline=init=/opt/scripts/tools/eMMC/init-eMMC-flasher-v3.sh' command is out of comment. This is a generic eMMC flasher provided by default.

2. Make sure that other required tools (**dosfsck, dosfslabel, rsync**) are available.

```
$ sudo apt install dosfstools rsync # install them if missing
```

3. When the board is booted from the uSD card, its eMMC device will be flashed with the uSD image automatically. The user LEDs will blink in the "cyclon" mode during flash process.

4. On completion of flash procedure the board will be powered down itself.

5. Remove the uSD card and restart the board with the 'POWER' button.

## How-to: Boot from the uSD card

1. Insert the uSD card into the card slot of BeagleBone Black.

2. Hold the 'S2' button down and power on the board. Once the user LEDs blick, release the 'S2' button.

3. Verify the serial device in the device list

```
$ ll /dev/ttyUSB0
```
4. Make a serial connection to the board

```
$ sudo screen /dev/ttyUSB0 115200
```

## How-to: Back up an actual image on a uSD card

```
$ sudo dd if=/dev/mmcblk0 of=sd_4g_beagleboneblack_archlinuxarm.img status=progress
```

## How-to: Connect the BeagleBone Black to a host serially

With adapter cable 'TTL-232R-3V3' connect the board to an USB port of the host. This connection will allow serial connection to the board.

# Source:
1. [BeagleBone Black Extracting eMMC contents](https://elinux.org/BeagleBone_Black_Extracting_eMMC_contents)
2. [Latest Firmware Images](https://beagleboard.org/latest-images)
