# Appendix: Limine Bootloader

This appendix is a swap-in replacement for the [core install guide](../arch-linux-install-guide.md)'s
section 11.0 (Install and Configure systemd-boot). Use it if you'd rather boot with Limine
instead of systemd-boot; everything else in the core guide continues unchanged.

**A note on the root device in `cmdline`:** the `cmdline` line below needs the actual device
path of your root filesystem, and that path depends on which disk layout you used:
- If you followed the core guide's plain partition layout, use `root=/dev/<your-root-partition>`
  (e.g. `root=/dev/nvme0n1p2` or `root=/dev/sda2` - whatever `lsblk` showed you).
- If you followed the [LVM appendix](lvm-disk-layout.md) instead, use `root=/dev/vg/root`.

## 1.0 Install limine
```shell
pacman -S limine
```

## 2.0 Install limine Bootloader
```shell
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
```

## 3.0 Create /boot/limine.conf
```shell
nano /boot/limine.conf
```

```conf
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/amd-ucode.img        # Remove if Intel, or if you skipped microcode
    module_path: boot():/intel-ucode.img      # Remove if AMD, or if you skipped microcode
    module_path: boot():/initramfs-linux.img
    cmdline: root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/amd-ucode.img        # Remove if Intel, or if you skipped microcode
    module_path: boot():/intel-ucode.img      # Remove if AMD, or if you skipped microcode
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Replace `root=/dev/vg/root` in both `cmdline` lines with whichever root device applies to your
setup, per the note at the top of this appendix. The `module_path` lines for microcode only
apply if you installed `amd-ucode`/`intel-ucode` from the
[graphics-and-extras appendix](graphics-and-extras.md); remove whichever line(s) don't apply.

## 4.0 Fix /boot Permissions
```shell
chmod 755 /boot
chmod 600 /boot/limine.conf
```
