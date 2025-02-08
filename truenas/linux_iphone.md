# Transfer files from iPhone 11 to a SMB share (TrueNAS 23.10)

## 1. Introduction

Instructions on how to transfer files from iPhone to a SMB shared storage via Linux.

   - Linux host: **Ubuntu 24.04**
   - iPhone 11: **iOS 17.6**
   - SMB share: **TrueNAS 23.10**

## 2. Install required packets

```
$ sudo apt install libimobiledevice6 libimobiledevice-utils ifuse gvfs-backends
[sudo] password for <user>:
...
libimobiledevice6 is already the newest version (1.3.0-8.1build3).
libimobiledevice6 set to manually installed.
gvfs-backends is already the newest version (1.54.0-1ubuntu2).
gvfs-backends set to manually installed.
The following additional packages will be installed:
  libfuse2t64
The following NEW packages will be installed:
  ifuse libfuse2t64 libimobiledevice-utils
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 196 kB of archives.
After this operation, 828 kB of additional disk space will be used.
Do you want to continue? [Y/n]
...

```

## 3. Connect iPhone to the Linux host

Connect the iPhone device to the linux host using an USB cable.
Choose 'Trust' if iPhone notify the connection.

Now check the connection:

```
$ lsusb
...
Bus 001 Device 006: ID 05ac:12a8 Apple, Inc. iPhone 5/5C/5S/6/SE/7/8/X/XR
...

$ idevicepair pair
SUCCESS: Paired with device e72f0c60056b489932eb29c77dddd6288b3fd96c
```

## 4. Mount iPhone using ifuse

```
$ mkdir ~/iphone
$ ifuse ~/iphone/    # unmount with '$ umount ~/iphone'
```

One can open 'Files' app to check the mounted folder.

## 5. Mount SMB shared storage

```
$ sudo mount -t cifs //<nas_server>/<share> ~/mnt/smb -o username=<user>,password=<password>,user,nounix,noperm
```

   - nas_server: **host_name** or **IP address** of the NAS server, ie., 192.168.1.188
   - share: name of the shared storage, ie., **ipple**


## 6. Transfer files from iPhone to the SMB share

Transfer files from iPhone to the shared storage using **rsync** command:

```
$ rsync -Pauvh ~/iphone/DCIM ~/mnt/smb/<user_iphone_model>/
```

## Issue#1: can not mount the SMB share with credentials:

Below command returns **error -13** (permission):

```
$ sudo mount -t cifs //192.168.1.188/ipple ~/mnt/smb -o credentials=~/.homenas_smb_credentials,user,nounix,noperm
```

But **~/.homenas_smb_credentials** has permission 600.
