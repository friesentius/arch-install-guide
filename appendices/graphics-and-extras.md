# Appendix: Graphics and Extras

This appendix covers steps that aren't required to reach a bootable base system, but that most
desktop/gaming setups will want next: CPU microcode updates, graphics drivers (with 32-bit
support for things like Steam), and the build tools needed to compile AUR packages. Run these
from inside the chroot, same as the core guide's Configure the System steps, or after your first
boot - either works.

## 1.0 Install Microcode
CPU microcode updates fix CPU-level bugs and security issues below the OS level; install the
one matching your CPU vendor - check `lscpu` if you're not sure. Pick one:

#### For AMD:
```shell
pacman -S amd-ucode
```

#### For Intel:
```shell
pacman -S intel-ucode
```

#### Regenerate initramfs to include microcode:
```shell
mkinitcpio -p linux
```
Microcode has to be loaded very early at boot, before the kernel proper starts, so it ships as
an extra image the initramfs loads - this rebuilds the initramfs to pick it up.

#### Then tell your bootloader about it:
- **systemd-boot** (core guide): add `initrd /amd-ucode.img` or `initrd /intel-ucode.img` as an
  extra `initrd` line, above the `initrd /initramfs-linux.img` line, in each of your
  `/boot/loader/entries/*.conf` files.
- **Limine** ([appendix](limine-bootloader.md)): already handled by the `module_path` lines in
  `limine.conf` - just remove the line for the vendor you don't have.

## 2.0 Install Graphics Drivers
#### Intel/AMD:
```shell
pacman -S mesa
```
`mesa` provides the open-source graphics drivers (OpenGL/Vulkan) for Intel and AMD GPUs. If you
have an Nvidia GPU, you'd install one of the `nvidia*` packages instead - out of scope for this
appendix.

#### Enable multilib:
```shell
nano /etc/pacman.conf
```
The `multilib` repository provides 32-bit versions of packages, needed for some 32-bit software
and games to run correctly on a 64-bit system. Uncomment (or add) this section:

```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Contains Steam and other 32-bit-dependent software.

##### Update:
```shell
pacman -Syu
```
Refreshes package databases (now including `multilib`) and upgrades installed packages.

## 3.0 Required to Install AUR Packages (optional)
```shell
pacman -S binutils make gcc pkg-config fakeroot debugedit git
```
The AUR (Arch User Repository) distributes build recipes (`PKGBUILD`s), not prebuilt binaries -
installing an AUR package means compiling it locally. This installs the common build toolchain
(compiler, linker, packaging tools) most AUR `PKGBUILD`s expect to find, plus `git` to fetch
them.
