# 7.0 Install Essential Packages: Clipboard Support

**Recommended default: `gpm`** - mouse-driven copy/paste on a plain console, this guide's
baseline. Add `wl-clipboard` or `xclip` instead only if you already know you're adding a Wayland
or X11 graphical session later; skip this doc entirely if you don't need clipboard support yet.

```shell
pacstrap /mnt gpm
systemctl enable --root=/mnt gpm
```
Installs the General Purpose Mouse daemon and enables it on the new system, so after reboot a
left-button drag selects text on any virtual console and a middle-click (right-click on a
two-button mouse) pastes it. `--root=/mnt` enables the service directly on the target filesystem
without chrooting first.

The live ISO already has `gpm` installed, just not running:
```shell
gpm
```
Starts it so you can select a long value like a disk UUID in one terminal and middle-click to
paste it into another.

**Adding a graphical session later:** install the matching clipboard tool when you set it up,
not now:
```shell
sudo pacman -S wl-clipboard   # Wayland
sudo pacman -S xclip          # X11
```

Continue at [8.0 Generate fstab](../arch-linux-install-guide.md#80-generate-fstab).
