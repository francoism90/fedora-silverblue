# Fedora Silverblue - Postinstall

Personal settings for Fedora Silverblue 39, with lovely help from:

- <https://rpmfusion.org/Howto/OSTree>
- <https://github.com/iaacornus/silverblue-postinstall_upgrade>

## Maintenance

```bash
rpm-ostree upgrade
flatpak repair
flatpak uninstall -y --unused
flatpak update -y --appstream
```

## RPM Fusion

RPM Fusion provides software that the Fedora Project or Red Hat doesn't want to ship.
See <https://rpmfusion.org/RPM%20Fusion> for details.

Add the repositories:

```bash
rpm-ostree install https://mirrors.rpmfusion.org/free/fedora/rpmfusion-free-release-$(rpm -E %fedora).noarch.rpm
rpm-ostree install https://mirrors.rpmfusion.org/nonfree/fedora/rpmfusion-nonfree-release-$(rpm -E %fedora).noarch.rpm
systemctl reboot
```

> **NOTE:** It is important to reboot first before continuing with the guide.

### Upgrading

To upgrade on major releases, e.g. Fedora Silverblue 38 > 39:

```bash
rpm-ostree update --uninstall rpmfusion-free-release --uninstall rpmfusion-nonfree-release \
    --install rpmfusion-free-release --install rpmfusion-nonfree-release
```

If you ever get in trouble on upgrading, you can reset the current ostree (create a backup first):

```bash
rpm-ostree status
rpm-ostree reset
systemctl reboot
```

Redo giving steps and reinstall your packages.

## Hardware acceleration

> **NOTE:** Make sure RPM Fusion is configured first.

See <https://rpmfusion.org/Howto/OSTree> when using Intel/NVIDIA, and require Broadcom/DVB firmwares.

### AMDGPU

Override current mesa-va-drivers:

```bash
rpm-ostree override remove mesa-va-drivers --install mesa-va-drivers-freeworld
```

In most cases you don't need VDPAU anymore, but if you do require it:

```bash
rpm-ostree install mesa-vdpau-drivers-freeworld
```

### FFMpeg

```bash
rpm-ostree install libavcodec-freeworld libva-utils
```

After a reboot, you can check support hardware decoding profiles using `vainfo`:

```bash
Trying display: wayland
libva info: VA-API version 1.20.0
libva info: Trying to open /usr/lib64/dri/radeonsi_drv_video.so
libva info: Found init function __vaDriverInit_1_20
libva info: va_openDriver() returns 0
vainfo: VA-API version: 1.20 (libva 2.20.1)
vainfo: Driver version: Mesa Gallium driver 23.3.1 for AMD Radeon Graphics (radeonsi, renoir, LLVM 17.0.6, DRM 3.54, 6.6.8-200.fc39.x86_64)
vainfo: Supported profile and entrypoints
      VAProfileMPEG2Simple            : VAEntrypointVLD
      VAProfileMPEG2Main              : VAEntrypointVLD
      VAProfileVC1Simple              : VAEntrypointVLD
      VAProfileVC1Main                : VAEntrypointVLD
      VAProfileVC1Advanced            : VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointVLD
      VAProfileH264ConstrainedBaseline: VAEntrypointEncSlice
      VAProfileH264Main               : VAEntrypointVLD
      VAProfileH264Main               : VAEntrypointEncSlice
      VAProfileH264High               : VAEntrypointVLD
      VAProfileH264High               : VAEntrypointEncSlice
      VAProfileHEVCMain               : VAEntrypointVLD
      VAProfileHEVCMain               : VAEntrypointEncSlice
      VAProfileHEVCMain10             : VAEntrypointVLD
      VAProfileHEVCMain10             : VAEntrypointEncSlice
      VAProfileJPEGBaseline           : VAEntrypointVLD
      VAProfileVP9Profile0            : VAEntrypointVLD
      VAProfileVP9Profile2            : VAEntrypointVLD
      VAProfileNone                   : VAEntrypointVideoProc
```

Generally you should see a lot more decoding and encoding support.

### GStreamer

```bash
rpm-ostee install rpm-ostree install gstreamer1-plugins-bad-free-extras gstreamer1-plugins-bad-freeworld gstreamer1-plugins-ugly gstreamer1-vaapi
```

## Filesystems

### Encryption

If you are using encryption on a NVMe/SSD, you may want to improve performance by disabling the workqueue.

See <https://wiki.archlinux.org/title/Dm-crypt/Specialties#Disable_workqueue_for_increased_solid_state_drive_(SSD)_performance> for details.

### Btrfs

If you are using Btrfs, you may want to enable <https://github.com/kdave/btrfsmaintenance>:

```bash
rpm-ostree install btrfsmaintenance
nano /etc/sysconfig/btrfsmaintenance
```

Enable the services:

```bash
sudo systemctl enable btrfs-balance.timer btrfs-defrag.timer btrfs-scrub.timer btrfs-trim.timer --now
```

## Software

It is discourage to install (large) software on the ostree. Try to use Flatpaks and your Toolbox (`toolbox enter`) as much as possible.

You can pull the latest toolbox, using:

```bash
podman pull fedora-toolbox:39
```

To update the Toolbox:

```bash
toolbox enter
sudo dnf update && sudo dnf upgrade
```

You can create multiple toolboxes, and even manage using [Podman Desktop](https://podman-desktop.io/).

### Replace Firefox

```bash
rpm-ostree override remove firefox firefox-langpacks
```

> **TIP:** The Mozilla Firefox Flatpak package is a good replacement.

> **TIP:** You can also hide the desktop entry itself.

#### Podman

If you use Podman, and need Docker compatibility:

```bash
rpm-ostree install podman-compose podman-docker
```

If possible, use rootless: <https://wiki.archlinux.org/title/Podman#Rootless_Podman>

### Theming

> **TIP:** See <https://itsfoss.com/flatpak-app-apply-theme/> instructions for Flatpak theming.

```bash
rpm-ostree install gnome-tweak-tool
```

When using GTK-3 apps, see <https://github.com/lassekongo83/adw-gtk3> for details.

```bash
flatpak install org.gtk.Gtk3theme.adw-gtk3 org.gtk.Gtk3theme.adw-gtk3-dark
```

After a reboot, you can apply theme settings using Gnome Tweaks.

### ZSH

See <https://ohmyz.sh/> for details.

```bash
rpm-ostree install fira-code-fonts fzf google-roboto-fonts mozilla-fira-mono-fonts powerline-fonts pygmentize tmux zsh zsh-autosuggestions zsh-syntax-highlighting
```
