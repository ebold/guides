# Launch BeagleBone Black with Debian Bookworm 12.2 IoT

Requires:
- 4GB/8GB uSD card (FAT formatted)
- 5V DC power adapter
- (optional) TTL-to-RS232 adapter cable

This short instruction shows how to run the BeagleBone Black board with Debian Bookworm 12.2 image.
For full or detailed instruction please refer to another instruction specified in [1] (or [click](launch_beaglebone_with_debian_buster.md) here).

## Back up the content of the eMMC device

This step is intentionally ignored.
If required please refer to [1].

## Prepare uSD card with new image

1. Download uSD image [AM335x 12.2 2023-10-07 4G eMMC IoT Flasher](https://www.beagleboard.org/distros)

2. Verify checksum

```
$ sha256sum am335x-eMMC-flasher-debian-12.2-iot-armhf-2023-10-07-4gb.img.xz
45e90c30917e5ae34c9d7b2b655933e5b054e65890d6cf1d61a3eea8c84c6da0  am335x-eMMC-flasher-debian-12.2-iot-armhf-2023-10-07-4gb.img.xz
```

3. Uncompress the image and write to the uSD card

```
$ unxz am335x-eMMC-flasher-debian-12.2-iot-armhf-2023-10-07-4gb.img.xz
$ sudo dd if=am335x-eMMC-flasher-debian-12.2-iot-armhf-2023-10-07-4gb.img of=/dev/mmcblk0 status=progress
```

## Flash the eMMC device with the uSD card image

1. Insert the uSD card into the card slot of BeagleBone Black.

2. Hold the 'S2' button down and power on the board. Once the user LEDs blink, release the 'S2' button.

3. On-board eMMC device will be flashed with the uSD image automatically. The user LEDs will blink in the "cyclon" mode during flash process.

4. On completion of flash procedure the board will be powered down itself.

5. Remove the uSD card.

## Re-boot from eMMC and log in to the board (via terminal emulator)

1. Connect the BeagleBone Black to a host serially

Use the adapter cable 'TTL-232R-3V3' to connect the board to an USB port of the host.

2. Verify the serial device in the device list

```
$ ll /dev/ttyUSB0
```

3. Launch terminal emulator (ie., minicom)

```
$ sudo minicom -D /dev/ttyUSB0 -b 115200
```

4. Power on the board. When boot completes the login prompt will be displayed.
Use the default login credential:

```
username:password is [debian:temppwd]
```

5. If keyboard is not reacting, disable the 'hardware flow control' of minicom (CTRL+A Z -> O -> Serial Port setup).

# Source:

1. [Launch BeagleBone Black with Debian Buster 10](launch_beaglebone_with_debian_buster.md)
