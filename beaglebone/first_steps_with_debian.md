# First steps after booting fresh Debian snapshot on BeagleBone Black

When you are logged in with the default user account, immediatelly perform at least following administration tasks.

## Change the default debian user password

Invoke 'passwd' to change the default password:

```
debian@BeagleBone:~$ passwd
Changing password for debian.
Current password:            # type here 'temppwd'
New password:
Retype new password:
passwd: password updated successfully
```

## Create a new user (and add it to sudoers)

Invoke 'useradd' to add a new user with following options:
- m: create the user account's home directory
- s: account's shell (the default shell for Ubuntu is /bin/sh)

Change 'ebob' with your desired user name. Afterwards, set password.

```
$ sudo useradd ebob -m -s /bin/bash

$ sudo passwd ebob
[sudo] password for debian:
New password:
Retype new password:
passwd: password updated successfully

$ ls -a ebob
.  ..  .bash_logout  .bashrc  .profile
```

Finally add new user to th 'sudo' group:

```
$ sudo usermod -aG sudo ebob
```

## (optional) Creat a superuser password

```
$ sudo su
# passwd
```

## (optional) Change the default settings of the on-board LEDs

Invoke a following command to check the actual trigger settings of an on-board LED (ie., usr0):

```
$ cat /sys/class/leds/beaglebone\:green\:usr0/trigger
```

Iterate over all LEDs by editing the LED index (usr0-3).
In Debian Bookworm the default trigger settings are specified in '/opt/source/dtb-5.10-ti/src/arm/am335x-bone-common.dtsi':

LED | trigger
--- | ---
usr0 | heartbeat
usr1 | mmc0
usr2 | cpu0
usr3 | mmc1

To set the trigger state of an user LED (ie, usr1) to 'none':

```
$ sudo echo none > /sys/class/leds/beaglebone\:green\:usr1/trigger
```

To change the default trigger settings permanently (invoke commands as **'debian'** user):
- edit the device tree source file:

```
$ vi /opt/source/dtb-5.10-ti/src/arm/am335x-bone-common.dtsi
```

- build the device tree binary and install it:

```
$ cd /opt/source/dtb-5.10-ti/
$ make
$ sudo make install
```

- reboot the system

```
$ sudo reboot
```

## Remove/Edit the login message [2]

The contents of the files '/etc/motd', '/etc/issue' and '/etc/issue.net' are displayed everytime at login.
In general, it's recommended to move these files elsewhere to remove the login message ('Beyond the Basics' in [2]):

```
$ sudo mv /etc/motd /etc/motd.orig
$ sudo mv /etc/issue /etc/issue.orig
$ sudo mv /etc/issue.net /etc/issue.net.orig
```

The latter 2 files include a line with the default login user credential (ie., debian:temppwd).
Just make backups and remove only undesired lines from the original issue files:

```
$ sudo cp /etc/issue /etc/issue.orig
$ sudo cp /etc/issue.net /etc/issue.net.orig
$ sudo vi /etc/issue              # remove a line with the default user credential
$ sudo vi /etc/issue.net
```

# Source:

1. [Getting Started with BeagleBone Black](https://community.element14.com/products/devtools/single-board-computers/next-genbeaglebone/b/blog/posts/getting-started-with-beaglebone-black)
2. [BeagleBone Cookbook](https://docs.beagleboard.org/latest/books/beaglebone-cookbook/)
