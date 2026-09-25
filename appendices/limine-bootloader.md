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
Installs the Limine bootloader package, providing its UEFI boot binary and supporting files.

## 2.0 Install limine Bootloader
```shell
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
```
Copies Limine's UEFI executable to the standard fallback boot path on the EFI partition
(`EFI/BOOT/BOOTX64.EFI`), which most UEFI firmware will boot automatically without needing a
separate boot-manager entry registered.

## 3.0 Create /boot/limine.conf
```shell
nano /boot/limine.conf
```
`limine.conf` is Limine's own boot menu configuration - it defines each bootable entry, which
kernel/initramfs to load, and the kernel command line.

```conf
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/amd-ucode.img        # Remove if Intel, or if you skipped microcode
    module_path: boot():/intel-ucode.img      # Remove if AMD, or if you skipped microcode
    module_path: boot():/initramfs-linux.img
    cmdline: root=/dev/<your-root-partition-or-vg-root> rw rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/amd-ucode.img        # Remove if Intel, or if you skipped microcode
    module_path: boot():/intel-ucode.img      # Remove if AMD, or if you skipped microcode
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: root=/dev/<your-root-partition-or-vg-root> rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Replace `/dev/<your-root-partition-or-vg-root>` in both `cmdline` lines with whichever root
device applies to your setup, per the note at the top of this appendix (a real partition path
like `/dev/nvme0n1p2`, or `/dev/vg/root` if you used the LVM appendix) - not the literal
placeholder text. The `module_path` lines for microcode only apply if you installed
`amd-ucode`/`intel-ucode` from the [graphics-and-extras appendix](graphics-and-extras.md);
remove whichever line(s) don't apply to your CPU vendor, or both if you skipped microcode
entirely.

## 4.0 Fix /boot Permissions
```shell
chmod 755 /boot
chmod 600 /boot/limine.conf
```
Restricts `limine.conf` to root-only reading (it can contain kernel command-line details you
may not want other local users to see) while keeping `/boot` itself traversable.
