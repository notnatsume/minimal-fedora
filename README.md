This guide will walk you through setting up Fedora Linux with the minimal GNOME desktop environment and additional packages according to your needs. The goal is a minimal yet functional system.

> You must disable Secure Boot in your BIOS/UEFI settings for Nvidia drivers to work. I will add a guide for Secure Boot support sometime later.

## Prerequisites
1. Download the **Fedora Everything Network Installer ISO** from [Alt Fedoraproject](https://alt.fedoraproject.org/).
2. Create a bootable USB drive using [Balena Etcher](https://etcher.balena.io/) and boot from it.
3. During installation:
   - Do minimal install by leaving package selection at default (Base Environment as `Fedora custom operating system` and select `Standard` in Additional Software Section).
   - Configure language, timezone, network, root password, and user account according to your preference.
   - Ensure an ethernet connection or use USB tethering as in some PC's WiFi will not work in the tty terminal post-install.

## Step-by-Step Setup

### 1. Optimize DNF Configuration
To make package installation faster and bit easy to use, configure DNF settings.

```bash
sudo nano /etc/dnf/dnf.conf
```

Add the following lines to the file:
```ini
max_parallel_downloads=10
defaultyes=True
fastestmirror=True
```

- `max_parallel_downloads=10`: Speeds up downloads by allowing up to 10 simultaneous connections.
- `defaultyes=True`: Defaults to "yes" for DNF prompts (skip if you prefer otherwise).
- `fastestmirror=True`: Picks the mirror with the lowest latency (may not provide fastest download speeds necessarily).

Save and exit (`Ctrl+O`, `Enter`, `Ctrl+X`).

### 2. Install RPM Fusion Repositories
RPM Fusion provides additional packages, including Nvidia drivers and Codecs.

```bash
sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
sudo dnf config-manager setopt fedora-cisco-openh264.enabled=1
```

### 3. Install Essential Packages
Install core utilities and development tools required for the system and drivers.

```bash
sudo dnf install -y curl wget git kernel-devel kernel-headers gcc make dkms acpid libglvnd-glx libglvnd-opengl libglvnd-devel pkgconfig unzip p7zip p7zip-plugins unrar
```

- `curl`, `wget`, `git`: Basic networking and version control tools.
- `kernel-devel`, `kernel-headers`, `gcc`, `make`, `dkms`: Required for driver compilation.
- `acpid`: Handles power events.
- `libglvnd*`, `pkgconfig`: Graphics and development libraries.
- `unzip`, `p7zip`, `unrar`: Archive utilities.

### 4. Set Up Flatpak and Flathub
Flatpak allows installing latest applications in a sandboxed environment.

```bash
sudo dnf install -y flatpak
sudo flatpak remote-add --if-not-exists flathub https://dl.flathub.org/repo/flathub.flatpakrepo
```

### 5. Install Minimal GNOME Desktop
Install only the core GNOME components for a lightweight desktop experience.

```bash
sudo dnf install -y gnome-shell gnome-tweaks gnome-terminal gnome-software nautilus NetworkManager-wifi gvfs-mtp
```

- `gnome-shell`: Core GNOME desktop.
- `gnome-tweaks`: Customization tool.
- `gnome-terminal`: Terminal emulator.
- `gnome-software`: GUI package manager.
- `nautilus`: File manager.
- `NetworkManager-wifi`: WiFi support.
- `gvfs-mtp`: Support for connecting mobile devices.

### 6. Install Codecs and Media Player
Add multimedia support for audio and video playback.

```bash
sudo dnf install -y ffmpeg --allowerasing
sudo dnf install -y ffmpeg-libs libva libva-utils
sudo dnf install -y celluloid
```

- `ffmpeg`, `ffmpeg-libs`: Handle most audio/video formats.
- `libva`, `libva-utils`: Hardware-accelerated video decoding.
- `celluloid`: Lightweight media player.
- `--allowerasing`: Resolves potential package conflicts.

### 7. Install Essential Fonts
Adds a selection of clean, modern fonts for better text rendering.

```bash
sudo dnf install -y google-roboto-fonts google-roboto-mono-fonts google-noto-sans-fonts google-noto-serif-fonts google-noto-mono-fonts dejavu-sans-fonts dejavu-serif-fonts dejavu-sans-mono-fonts liberation-sans-fonts liberation-serif-fonts liberation-mono-fonts
```

> **Note**: The fonts installed above are Latin-based and may not support non-Latin scripts like Arabic, Chinese, Japanese, Hindi and others. To add support for regional languages, install the appropriate fonts based on your needs by checking here: [Fonts List](https://packages.fedoraproject.org/pkgs/google-noto-fonts/).

Else install fonts for all languages(scripts).

```bash
sudo dnf install -y google-noto-fonts-all
```

### 8. Install Librewolf Browser
Librewolf is a privacy-focused fork of Firefox, installed via Flatpak.

```bash
flatpak install flathub io.gitlab.librewolf-community
```

### 9. Install Nvidia Drivers
If you have an Nvidia GPU, install the proprietary drivers and related tools.

```bash
sudo dnf install -y akmod-nvidia xorg-x11-drv-nvidia-cuda
sudo dnf install -y nvidia-vaapi-driver libva-utils vdpauinfo libva-nvidia-driver
```

> **Note**: Wait for 5-10 minutes after installation for the `akmod-nvidia` kernel module to compile before rebooting.

### 10. Clean Up GRUB Bootloader (Optional)
Make the boot process cleaner and quieter by editing GRUB settings.

```bash
sudo nano /etc/default/grub
```

Modify or add the following lines:
```ini
GRUB_TIMEOUT=0
GRUB_TIMEOUT_STYLE=hidden
GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"
```

- `GRUB_TIMEOUT=0`: Hides the GRUB menu (set to 5-10 seconds for dual-boot systems).
- `GRUB_TIMEOUT_STYLE=hidden`: Skips the GRUB countdown.
- `quiet splash`: Reduces boot messages and shows a splash screen.

Next modify `GRUB_CMDLINE_LINUX` by removing `rhgb` and adding `loglevel=0`:

Update GRUB:
```bash
sudo grub2-mkconfig -o /boot/grub2/grub.cfg
```

### 11. Set GNOME as Default
Switch from the console to the GNOME graphical environment.

```bash
sudo systemctl set-default graphical.target
```

### 12. Install Visual Studio Code or VSCodium (Optional)
Code editor that can be used with a variety of programming languages.

```bash
sudo rpm --import https://packages.microsoft.com/keys/microsoft.asc
echo -e "[code]\nname=Visual Studio Code\nbaseurl=https://packages.microsoft.com/yumrepos/vscode\nenabled=1\nautorefresh=1\ntype=rpm-md\ngpgcheck=1\ngpgkey=https://packages.microsoft.com/keys/microsoft.asc" | sudo tee /etc/yum.repos.d/vscode.repo > /dev/null
sudo dnf check-update
sudo dnf install code
```

> If you prefer a telemetry-free code editor, install VSCodium (a fork of VS Code).

```bash
sudo rpmkeys --import https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg
printf "[gitlab.com_paulcarroty_vscodium_repo]\nname=download.vscodium.com\nbaseurl=https://download.vscodium.com/rpms/\nenabled=1\ngpgcheck=1\nrepo_gpgcheck=1\ngpgkey=https://gitlab.com/paulcarroty/vscodium-deb-rpm-repo/-/raw/master/pub.gpg\nmetadata_expire=1h\n" | sudo tee -a /etc/yum.repos.d/vscodium.repo
sudo dnf check-update
sudo dnf install codium
```

### 13. Install Android Tools (Optional)
For Android development or debugging (adb).

```bash
sudo dnf install -y android-tools
```

### 14. Install Gnome Image Viewer (Optional)
Minimal and GPU accelerated image viewer by GNOME Project.

```bash
flatpak install flathub org.gnome.Loupe
```

### 15. Install GNOME Extension Manager (Optional)
Manage and install GNOME extensions easily with this Flatpak app.

```bash
flatpak install flathub com.mattjakeman.ExtensionManager
```

### 16. Install GNOME File Roller (Optional)
Create and view archive files, such as tar, zip, 7zip, rar and others.

```bash
sudo dnf install file-roller
```

### 17. Install GNOME Disk Utility (Optional)
For partitioning, file system creation and other features.

```bash
sudo dnf install gnome-disk-utility
```

### 18. Install GNOME Calendar (Optional)
Create and manage calendars and events.

```bash
flatpak install flathub org.gnome.Calendar
```

### 19. Install GNOME Contacts (Optional)
Keep and organize your contacts information.

```bash
flatpak install flathub org.gnome.Contacts
```

### 20. Install GNOME Weather (Optional)
Monitor the weather conditions of your city, or anywhere in the world.

```bash
flatpak install flathub org.gnome.Weather
```

### 21. Install GNOME Text Editor (Optional)
Simple text editor, similar to windows notepad.

```bash
flatpak install flathub org.gnome.TextEditor
```

### 22. Install GNOME Music Player (Optional)
Audio player that just plays audio files.

```bash
flatpak install flathub org.gnome.Decibels
```

> This app is very minimal, offering only basic functionality. If you want more features, install the app below.

```bash
flatpak install flathub org.gnome.Music
```

### 23. Install an Office Suite - OnlyOffice or LibreOffice (Optional)
If you are coming from Microsoft Office, OnlyOffice is the perfect alternative.

```bash
flatpak install flathub org.onlyoffice.desktopeditors
```

> If you want a more powerful office suite, go with LibreOffice.

```bash
flatpak install flathub org.libreoffice.LibreOffice
```

### 24. Install Resources Monitor (Optional)
Monitor your CPU, Memory, Disk, Network and GPU usage. Similar to task manager on windows.

```bash
flatpak install flathub io.missioncenter.MissionCenter
```

### 25. Reboot
Restart to load the new GNOME desktop.

```bash
sudo reboot now
```

---

### Welcome to the world of Linux!

One of the greatest strengths of Linux is freedom. You have complete control over your system from the desktop environment to individual software packages.

Unlike Windows or macOS, which come with a set of unremovable pre-installed packages, linux lets you choose exactly what you want and leave out what you don't.

Enjoy your Fedora GNOME setup!
