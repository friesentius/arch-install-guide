# 7.0 Install Essential Packages: Clipboard Support

**Default: `gpm`** - mouse-driven copy/paste on a plain console, this guide's baseline.

```shell
pacstrap /mnt gpm
systemctl enable --root=/mnt gpm
```
Installs the General Purpose Mouse daemon and enables it on the new system, so after reboot a
left-button drag selects text on any virtual console and a middle-click (right-click on a
two-button mouse) pastes it, including into a different console. `--root=/mnt` enables the
service directly on the target filesystem without chrooting first.

**Adding a graphical session later:** install the matching clipboard tool then, not now:
```shell
sudo pacman -S wl-clipboard   # Wayland
sudo pacman -S xclip          # X11
```

Continue at [8.0 Generate fstab](../arch-linux-install-guide.md#80-generate-fstab).
