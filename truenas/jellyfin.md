# Mount SMB share (TrueNAS Scale 25.04)

## 1. Set up dedicated datasets

Add a new dataset in 'Datasets':

   - Parent Path = primary/apps
   - Name = jellyfin
   - Dataset Preset = **Apps**

Repeat it and create 2 other sub-datasets in 'primary/apps/jellyfin/'
   - cache
   - config

An SMB share 'primary/ipple' will be used as media library by jellyfin. In order to let jellyfin access to it, add 'apps' built-in group to the permission (ACL) list of this shared folder:
   - Who = Group
   - Group = apps
   - ACL Type = allow
   - Permissions Type = Basic
   - Permissions = Read
   - Flags Type = Basic
   - Flags = Inherit

Before saving ACL set the following checkboxes:
   - [x] Apply permissions recursively
   - [x] Apply permissions to child datasets

## 2. Install jellyfin app

In the 'Apps' section find and install 'jellyfin' (v1.2.8, app version 10.10.7)

Important settings:

   - Jellyfin Configuration:
      - Published Server URL: **jellyfin host IP address**

   - Network Configuration:
      - Host Network = **checked**

   - Storage Configuration:
      - Jellyfin Config Storage: specify host path to **'/mnt/primary/apps/jellyfin/config'**
      - Jellyfin Cache Storage: specify host path to **'/mnt/primary/apps/jellyfin/cache'**
      - Jellyfin Transcode Storage:
         - Type = tmpfs
         - Tmpfs Size Limit (in Mi) = 500
      - Additional Storage:
         - /media - /mnt/primary/media (Type = Host Path)
         - /backup - /mnt/primary/home/Backup (Type = Host Path)
         - /ipple - /mnt/primary/ipple (Type = Host Path, **[x] Enable ACL, [x] Force Flag**)

With above settings the jellyfin server (v10.10.7) is accessible under the port number 8096 (e.g., http://jellyfin_host_ip_address:8096)

## 3. Setup the jellyfin server

On any web browser navigate to your jellyfin server: **jellyfin_host_ip_address:8096**

Use an existing user credential to log in and configure basic stuff.

Add media libraries in 'Libraries':
   - click on 'Add Media Library'
      - select the content type (e.g., 'Home Videos and Photos' for your private media)
      - change the 'Display name' if you want
      - add folders by specifying mounted storages (e.g., /media)
      - (optional) check both fields of 'Chapter Images'

The deployed libraries will be scanned automatically (you can also click on 'Scan all Libraries' for all libraries or Ampel-points for a specific library and choose 'Scan library')

## 4. View contents

Use web browser or jellyfin player (on Android TV 10)

## 5. Issues

### 5.1. Vertically recorded phone videos are rotated 90 degrees

If using jellyfin player on Android TV (v10), then the videos are rotated 90 degrees. Likely it's known issue ([jellyfin-androidtv: issue \#3934 ](https://github.com/jellyfin/jellyfin-androidtv/issues/3934)) and not yet solved.
