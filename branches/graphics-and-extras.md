# Branch: Graphics and Extras

The [main guide](../arch-linux-install-guide.md)'s 25.0 Graphics, Microcode, and AUR Tools: CPU
microcode, graphics drivers with 32-bit support, and the tools to build AUR packages. Run it
from your regular user. Commands are shown with `sudo`; if `which sudo doas 2>/dev/null` shows
only `doas`, use `doas` instead.

## 1.0 Install Microcode
Microcode updates fix CPU bugs and security issues. Install the one for your CPU vendor (`lscpu`
shows it):

#### For AMD:
```shell
sudo pacman -S amd-ucode
```

#### For Intel:
```shell
sudo pacman -S intel-ucode
```

#### Regenerate initramfs to include microcode:
```shell
sudo mkinitcpio -P
```
The `microcode` hook embeds the update in the initramfs, so it loads early at boot with no
bootloader changes needed.

## 2.0 Install Graphics Drivers
#### Intel/AMD:
```shell
sudo pacman -S mesa
```
`mesa` provides the open-source OpenGL/Vulkan drivers for Intel and AMD GPUs. Nvidia GPUs need
one of the `nvidia*` packages instead, which this guide doesn't cover.

#### Enable multilib:
```shell
sudo nano /etc/pacman.conf
```
The `multilib` repository holds 32-bit packages that Steam and some games need. Uncomment this
section:

```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Then refresh the package databases and upgrade:
```shell
sudo pacman -Syu
```

## 3.0 Required to Install AUR Packages (optional)
```shell
sudo pacman -S binutils make gcc pkg-config fakeroot debugedit git
```
The AUR (Arch User Repository) ships build recipes (`PKGBUILD`s), not binaries, so installing
from it means compiling locally. This installs the build toolchain most `PKGBUILD`s expect, plus
`git` to fetch them.

## Continue in the main guide
Continue at [26.0 Firewall](../arch-linux-install-guide.md#260-firewall).
