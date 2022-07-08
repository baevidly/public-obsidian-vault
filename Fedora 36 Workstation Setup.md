# DNF

Edit configuration: `sudo nano /etc/dnf/dnf.conf`

- Add settings to optimize download speed:
  ```conf
  max_parallel_downloads=20
  deltarpm=True
  ```
- Add some quality of life settings:
  ```conf
  defaultyes=True
  ```

More info at https://dnf.readthedocs.io/en/latest/conf_ref.html.

# RPM Fusion

- Add free RPM Fusion repositories:
    ```sh
    sudo dnf install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
    sudo dnf install https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
    ```
- Update AppStream metadata: `sudo dnf groupupdate core`

More info at https://rpmfusion.org/Configuration.

# Flatpak

- Add Flathub repository: `flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo`
- Disable default Fedora Flathub repository: `sudo flatpak remote-modify --disable fedora`
- Install Flatseal to control Flatpak apps permissions: `flatpak install flathub com.github.tchx84.Flatseal`
- Fix themes/icons/fonts in Flatpak apps:
    ```sh
    sudo flatpak override --filesystem=$HOME/.themes
    sudo flatpak override --filesystem=$HOME/.icons
    sudo flatpak override --filesystem=$HOME/.fonts
    sudo flatpak override --filesystem=xdg-config/gtk-3.0
    sudo flatpak override --filesystem=xdg-config/gtk-4.0
    ```
- Review your Flatpak overrides: `sudo flatpak override --show`

# Update System

- `sudo dnf upgrade --refresh`
- Reboot

# NVIDIA Drivers

- **Disable Secure Boot in your BIOS**
- Install and CUDA drivers (for latest drivers see the [[#Latest drivers]] section): `sudo dnf install akmod-nvidia xorg-x11-drv-nvidia-cuda`
- **Wait up to 5 minutes after the RPM transaction ends, until the kernel module get built**
- Check if kernel module is built: `modinfo -F version nvidia`
- Reboot

More info at https://rpmfusion.org/Howto/NVIDIA.

## Latest drivers

Install latest drivers from the [Rawhide](https://docs.fedoraproject.org/en-US/releases/rawhide/) repository:
```sh
sudo dnf install "kernel-devel-uname-r >= $(uname -r)"
sudo dnf update
sudo dnf copr enable kwizart/nvidia-driver-rawhide
sudo dnf install rpmfusion-nonfree-release-rawhide
sudo dnf --enablerepo=rpmfusion-nonfree-rawhide install akmod-nvidia xorg-x11-drv-nvidia xorg-x11-drv-nvidia-cuda --best
```

More info at https://rpmfusion.org/Howto/NVIDIA#Latest.2FBeta_driver.

## Fix hibernation

Edit configuration `sudo nano /etc/modprobe.d/nvidia.conf`:
```
options nvidia NVreg_PreserveVideoMemoryAllocations=0
```

Re-enable NVIDIA services:
```sh
sudo systemctl reenable nvidia-suspend.service nvidia-resume.service nvidia-hibernate.service
```

# Media Codecs

```sh
sudo dnf install gstreamer1-plugins-{bad-\*,good-\*,base} gstreamer1-plugin-openh264 gstreamer1-libav --exclude=gstreamer1-plugins-bad-free-devel
sudo dnf install lame\* --exclude=lame-devel
sudo dnf group upgrade --with-optional Multimedia
```

## Fix thumbnails
- Install `ffmpegthumbnailer`: `sudo dnf install ffmpegthumbnailer`
- Close `nautilus` and clear thumnails cache:
```sh
	nautilus -q
	rm -r ~/.cache/thumbnails/*
``` 

# Fonts

- Enable font hinting and antialiasing: 
    ```sh
    gsettings set org.gnome.desktop.interface font-hinting slight
    gsettings set org.gnome.desktop.interface font-antialiasing rgba
    ```
- *Optional*. Scale fonts: `gsettings set org.gnome.desktop.interface text-scaling-factor 1.1`
- *Optional.* Install popular fonts:
    ```sh
    sudo dnf install \
        fira-code-fonts \
        google-noto-sans-fonts \
        google-noto-serif-fonts \
        google-noto-sans-mono-fonts \
        'google-roboto*fonts' \
        'google-droid*fonts' \
        'mozilla-fira*fonts' \
        'adobe*pro-fonts' \
        'dejavu-sans*fonts' \
        'dejavu-serif*fonts' \
        fontawesome-fonts \
        levien-inconsolata-fonts \
        jetbrains-mono-fonts-all \
        ibm-plex-fonts-all \
        pt-sans-fonts \
        open-sans-fonts
    ```
- *Optional*. Install Ubuntu font: https://design.ubuntu.com/font/

## MS Fonts

Install dependencies and fonts:
```sh
sudo dnf install cabextract xorg-x11-font-utils
sudo rpm -i https://downloads.sourceforge.net/project/mscorefonts2/rpms/msttcore-fonts-installer-2.6-1.noarch.rpm
```

More info at http://mscorefonts2.sourceforge.net/.

# GNOME Customization

- Add empty file template for fast file creation in Nautilus: `touch "$HOME/Templates/Empty File"`
- Install GNOME Tweaks, the graphical interface for advanced GNOME settings: `sudo dnf install gnome-tweaks`
- *Optional.* For best look and consistency install GTK3 port of the [libadwaita](https://gnome.pages.gitlab.gnome.org/libadwaita/) theme using instructions at https://github.com/lassekongo83/adw-gtk3 (don't forget the [GTK4 part](https://github.com/lassekongo83/adw-gtk3#gtk-4))
- *Optional.* Install Papirus Icon Theme: `sudo dnf install papirus-icon-theme` or https://github.com/PapirusDevelopmentTeam/papirus-icon-theme
- Enable Lock Screen: `gsettings set org.gnome.desktop.lockdown disable-lock-screen false`

# GNOME Extensions

More info at https://docs.fedoraproject.org/en-US/quick-docs/gnome-shell-extensions/.

- Install Extension Manager: `flatpak install flathub com.mattjakeman.ExtensionManager`

Useful extensions:

- [AppIndicator and KStatusNotifierItem Support](https://extensions.gnome.org/extension/615/appindicator-support/): adds AppIndicator, KStatusNotifierItem and legacy Tray icons support to the GNOME Shell
- [Caffeine](https://extensions.gnome.org/extension/517/caffeine/): disable the screensaver and auto-suspend when needed
- [Clipboard Indicator](https://extensions.gnome.org/extension/779/clipboard-indicator/): adds a clipboard indicator to the top panel, and caches clipboard history
- [GSConnect](https://extensions.gnome.org/extension/1319/gsconnect/): allows devices to securely share content like notifications or files and other features like SMS messaging and remote control (also install `nautilus-python` for full support: `sudo dnf install nautilus-python`)
- [Hot Edge](https://extensions.gnome.org/extension/4222/hot-edge/): add a hot edge that activates the overview to the bottom of the screen
- [Legacy (GTK3) Theme Scheme Auto Switcher](https://extensions.gnome.org/extension/4998/legacy-gtk3-theme-scheme-auto-switcher/): change the GTK3 theme to light/dark variant based on the system color scheme on Gnome 42
- [Sound Input & Output Device Chooser](https://extensions.gnome.org/extension/906/sound-output-device-chooser/): shows a list of sound output and input devices (similar to gnome sound settings) in the status menu below the volume slider
- [Sound Percentage](https://extensions.gnome.org/extension/2120/sound-percentage/): display the current sound percentage in the system tray
- [Status Area Horizontal Spacing](https://extensions.gnome.org/extension/355/status-area-horizontal-spacing/): reduce the horizontal spacing between icons in the top-right status area
- [User Themes](https://extensions.gnome.org/extension/19/user-themes/): load shell themes from user directory
- [Vitals](https://extensions.gnome.org/extension/1460/vitals/): a glimpse into your computer's temperature, voltage, fan speed, memory usage, processor load, system resources, network speed and storage stats
- [Dash to Dock for COSMIC](https://extensions.gnome.org/extension/5004/dash-to-dock-for-cosmic/): moves the dash out of the overview transforming it in a dock for an easier launching of applications and a faster switching between windows and desktops
- [Gesture Improvements](https://extensions.gnome.org/extension/4245/gesture-improvements/): improve touchpad gestures
- [Just Perfection](https://extensions.gnome.org/extension/3843/just-perfection/): tweak tool to customize GNOME Shell
- [Tiling Assistant](https://extensions.gnome.org/extension/3733/tiling-assistant/): expand GNOME's column tiling and add a Windows-snap-assist-inspired popup
- [Tray Icons: Reloaded](https://extensions.gnome.org/extension/2890/tray-icons-reloaded/): bring back Tray Icons to top panel

# Applications

```
...
```

# Maintenance

- Update packages and flatpak apps:
    ```sh
    sudo dnf upgrade --refresh
    flatpak update
    ```
- Cleanup system:
    ```sh
    sudo dnf autoremove
    sudo dnf clean all
    flatpak uninstall --unused
    flatpak uninstall --delete-data
    ```

# Scrolling

From https://askubuntu.com/a/621140:

- Install `imwheel`: `sudo dnf install imwheel`
- Create `imwheel` config file: `nano ~/.imwheelrc`:
    ```
    ".*"
    None,      Up,   Button4, 2
    None,      Down, Button5, 2
    Control_L, Up,   Control_L|Button4
    Control_L, Down, Control_L|Button5
    Shift_L,   Up,   Shift_L|Button4
    Shift_L,   Down, Shift_L|Button5
    ```
- Start `imwheel` and add to `.bash_profile`: `imwheel -b 45 --kill`

# Kernel Rollback

- Enable previous menu entry saving in GRUB config: `sudo nano /etc/default/grub`
  ```conf
  GRUB_SAVEDEFAULT=true
  ```
- Generate GRUB config: `sudo grub2-mkconfig -o /boot/grub2/grub.cfg`

## Install older kernel

If there's no needed kernel listed in GRUB, you can install it manually from RPMs using `koji`:

- Download kernel RPMs: 
    ```sh
    mkdir ~/Downloads/kernel && cd ~/Downloads/kernel
    koji download-build --arch=x86_64 kernel-5.17.15-300.fc36
    ```
- Install kernel: `sudo dnf install kernel-5* kernel-core-5* kernel-modules-5* kernel-devel-5*`
- Rebuild kernel modules (like NVIDIA) and update boot image: `sudo akmods --force && sudo dracut --force`

## Lock kernel version
- Install the `versionlock` DNF pluigin: `sudo dnf install 'dnf-command(versionlock)'`
- Lock kernel version: `sudo dnf versionlock add kernel{,-modules,-core,-devel,-modules-extra}-5.17.15`

# Virtualization

## libvirtd

- Install virtualization tools: `sudo dnf install @virtualization`
- Enable and start `libvrtd`: `sudo systemctl enable libvirtd && sudo systemctl start libvirtd`

More info at https://docs.fedoraproject.org/en-US/quick-docs/getting-started-with-virtualization/.

## VirtualBox

- Add VirtualBox repo: `sudo dnf config-manager --add-repo=https://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo`
- Install VirtualBox: `sudo dnf install VirtualBox-6.1`

# Disable IPv6

- Create `sysctl` config file: `sudo nano /etc/sysctl.d/00-disable-ipv6.conf`
  ```conf
  net.ipv6.conf.all.disable_ipv6=1
  net.ipv6.conf.default.disable_ipv6=1
  net.ipv6.conf.lo.disable_ipv6=1
  ```
- Reload `sysctl` settings: `sudo sysctl -p`

# Steam Launch Options

- Civilization VI: `LD_PRELOAD=/usr/lib64/libfreetype.so.6 %command%`