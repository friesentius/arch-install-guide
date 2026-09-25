# Appendix: Graphics and Extras

This appendix covers steps that aren't required to reach a bootable base system, but that most
desktop/gaming setups will want next: CPU microcode updates, graphics drivers (with 32-bit
support for things like Steam), and the build tools needed to compile AUR packages. Run these
from inside the chroot, same as the core guide's Configure the System steps, or after your first
boot - either works.

## 1.0 Install Microcode
CPU microcode updates fix CPU-level bugs and should match your CPU vendor. Pick one:

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

#### Enable multilib:
```shell
nano /etc/pacman.conf
```

```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Contains steam, etc.

##### Update:
```shell
pacman -Syu
```

## 3.0 Required to Install AUR Packages (optional)
```shell
pacman -S binutils make gcc pkg-config fakeroot debugedit git
```
