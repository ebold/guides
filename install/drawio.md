# Installation of draw.io (AppImage) for all users

Last Updated: January 2026

Follow these steps to install draw.io globally in the `/opt` directory on Linux.

### ✅ Prerequisites
- [ ] Administrative (sudo) privileges.
- [ ] `libfuse2` installed (required for AppImages on modern distros).

---

### 🚀 Installation Steps

#### 1. Create the directory
Create a dedicated folder to house the binary:
```bash
sudo mkdir -p /opt/drawio

```

#### 2. Download the latest version
Download the AppImage directly from the official releases:

```bash
sudo wget https://github.com/jgraph/drawio-desktop/releases/download/v29.3.0/drawio-x86_64-29.3.0.AppImage -O /opt/drawio/drawio-x86_64-29.3.0.AppImage

```

#### 3. Set executable permissions
Ensure the file is executable for all users:

```bash
sudo chmod +x /opt/drawio/drawio-x86_64-29.3.0.AppImage
```

#### 4. Create desktop integration
Create a desktop entry so the app appears in the application launcher:
```bash
sudo nano /usr/share/applications/drawio.desktop
```

Paste the following content:
```ini
[Desktop Entry]
Name=draw.io
Exec=/opt/drawio/drawio-x86_64-29.3.0.AppImage
Icon=drawio
Type=Application
Categories=Graphics;Office;
```

Press _Ctrl+O_, _Enter_ to save, and _Ctrl+X_ to exit.

#### 5. Add an icon (optional but recommended)
If the icon does not appear automatically, you can download a draw.io icon and place it in the system icon folder:
```bash
sudo wget https://github.com/jgraph/drawio-desktop/blob/dev/build/512x512.png -O /usr/share/icons/hicolor/512x512/apps/drawio.png
sudo gtk-update-icon-cache /usr/share/icons/hicolor
```

#### 6. Important note for older systems (FUSE)
In 2026, most modern Linux distributions (like Ubuntu 24.04 or later) use **FUSE 3**.
However, AppImages often require **FUSE 2** to run. If the app fails to launch, install the legacy FUSE library:
   * Ubuntu/Debian/Mint: ```sudo apt install libfuse2```
   * Fedora: ```sudo dnf install fuse-libs```

### 🛠 Troubleshooting
```bash
sudo dpkg --list | grep -i fuse
ii  fuse3                                      3.10.5-1build1                             amd64        Filesystem in Userspace (3.x version)
ii  ifuse                                      1.1.4~git20181007.3b00243-1                amd64        FUSE module for iPhone and iPod Touch devices
ii  libfuse2:amd64                             2.9.9-5ubuntu3                             amd64        Filesystem in Userspace (library)
ii  libfuse3-3:amd64                           3.10.5-1build1                             amd64        Filesystem in Userspace (library) (3.x version)
```

### Used Github markdown features:

1.  **Task Lists (`- [ ]`):** Great for "Prerequisites" so users can track progress.
2.  **Code Blocks (triple backticks):** Syntax highlighting for `bash` and `ini` makes it easy to read and copy.
3.  **Admonitions (`> [!TIP]`):** A newer GitHub feature that creates a color-coded callout box for tips or warnings.
4.  **Horizontal Rules (`---`):** Used to visually separate different phases of the setup.
