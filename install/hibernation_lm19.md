# Enable 'hibernate' in Linux Mint 19 (LM19)

TODO: add description in mongolian

This guide will help you to set up working hibernation on a fresh install of Linux Mint 19.x.

## Background

In general hibernation writes the content of RAM into the swap space of your computer. On start, it re-loads the content from the swap space to resume. There are 2 ways to set up a swap space: use of __swap partition__ or __swap file__ (for LM 19.x or newer).
Here the former method is described: using a separate swap partition.

## Check a separate swap partition

In order to enable hibernation a **separate swap partition** (larger than RAM size) is required.

To determine the type of the swap space:
```
$ cat /proc/swaps

Filename    Type        Size       Used    Priority
/dev/sdb1   partition   17169404   0       -2             # swap partition
```

If the command output looks like below, then your system uses the swap file:
```
Filename    Type   Size      Used    Priority
/swapfile   file   2097148   38356   -2                   # swap file!
```

### Update fstab (if fstab does not contain swap partition)

Because of default swap space as swap file, the swap partition might not be listed in **/etc/fstab**. If so remove the swap file and add the swap partition to **/etc/fstab**:
```
UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx none            swap    sw              0       0     # entry with swap partition info
#/swapfile                                 none            swap    sw              0       0
```

## Check if hibernation works from CLI

Before enabling the hibernation option in the shutdown menu, one has to check if hibernate works at all with **systemdctl**.

```
$ systemctl hibernate      # your PC should shut down, if hibernate works!
```

## Set up system configuration

To enable the hibernate option certain configurations must be done.

### systemd configuration

This is the primary configuration that show the hibernate option in the shutdown menu. As root user you have to create **/etc/polkit-1/localauthority/50-local.d/com.ubuntu.enable-hibernate.pkla** file with the following content:

```
[Re-enable hibernate by default in upower]
Identity=unix-user:*
Action=org.freedesktop.upower.hibernate
ResultActive=yes

[Re-enable hibernate by default in logind]
Identity=unix-user:*
Action=org.freedesktop.login1.hibernate;org.freedesktop.login1.handle-hibernate-key;org.freedesktop.login1;org.freedesktop.login1.hibernate-multiple-sessions;org.freedesktop.login1.hibernate-ignore-inhibit
ResultActive=yes
```

A reboot is required to apply these settings. New hibernate option should appear in the shutdown menu.

### GRUB configuration (if LM19 cannot resume)

It might happen that LM19 cannot resume from hibernation. In this case resume option must be added into the GRUB configuration file, **/etc/default/grub**. For that you need to have the UUID of the swap partition.

```
$ sudo blkid | grep swap   # get UUID of the swap partition
```

Alter the boot option with **GRUB_CMDLINE_LINUX_DEFAULT**, so that it includes also **resume** option:

```
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash resume=UUID=xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

Finally update the GRUB configuration and initramfs files, and reboot your system:
```
$ sudo update-grub                              # update the GRUB configuration
$ sudo rm /etc/initramfs-tools/conf.d/resume    # remove unused resume file if exists (boost startup now)
$ sudo update-initramfs -u -k all               # update initramfs files
$ sudo reboot                                   # can be done later!
```

## External links

1. Tip from Ehlertronic, in [german](https://www.ehlertronic.de/linux/tipps-tricks/ruhezustand.html)
