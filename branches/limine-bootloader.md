# Branch: Limine Bootloader

This branch is a swap-in replacement for the [main guide](../arch-linux-install-guide.md)'s
20.0 Install and Configure systemd-boot step. Use it if you'd rather boot with Limine instead of
systemd-boot; everything else in the main guide continues unchanged.

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

#### Find your cmdline value
First, work out the `cmdline` value your entries need. Check your disk layout now with
`lsblk -f`: look for `crypto_LUKS` and/or `LVM2_member` in the output. If you see `crypto_LUKS`
(with or without `LVM2_member` alongside it), first find your root partition's UUID (the
underlying encrypted partition's UUID, not the mapper device's):
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```

**Plain partition layout (default):**
```
cmdline: root=/dev/<your-root-partition> rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Replace `/dev/<your-root-partition>` with the actual root partition device you formatted (e.g.
`/dev/nvme0n1p2` or `/dev/sda2`).

**LVM only:**
```
cmdline: root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none
```

**Full-disk encryption only:**
```
cmdline: cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/mapper/cryptroot rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Replace `<your-root-partition-uuid>` with the UUID `blkid` printed above. `cryptdevice=UUID=...
:cryptroot` tells the initramfs's `encrypt` hook which device to unlock and what to name the
resulting mapper device (`cryptroot`, matching what you named it when you ran `cryptsetup open`
while partitioning); `root=/dev/mapper/cryptroot` then points the kernel at the now-unlocked
device.

**LVM + full-disk encryption (LVM-on-LUKS):**
```
cmdline: cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Same `cryptdevice=` as above (replace `<your-root-partition-uuid>` with your `blkid` output),
but `root=` points at the LVM logical volume, which the initramfs's `lvm2` hook finds inside the
now-unlocked container.

#### Write the config
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
    cmdline: root=/dev/<your-root-partition> rw rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/amd-ucode.img        # Remove if Intel, or if you skipped microcode
    module_path: boot():/intel-ucode.img      # Remove if AMD, or if you skipped microcode
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: root=/dev/<your-root-partition> rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Replace both `cmdline` lines with whichever one you worked out above for your disk layout - not
the literal placeholder text shown here. The `module_path` lines for microcode only apply if you
installed `amd-ucode`/`intel-ucode` from the
[graphics-and-extras branch](graphics-and-extras.md); remove whichever line(s) don't apply to
your CPU vendor, or both if you skipped microcode entirely.

## 4.0 Fix /boot Permissions
```shell
chmod 755 /boot
chmod 600 /boot/limine.conf
```
Restricts `limine.conf` to root-only reading (it can contain kernel command-line details you
may not want other local users to see) while keeping `/boot` itself traversable.

## Continue in the main guide
Your bootloader is set up. Continue with the main guide's
[21.0 Exit Chroot](../arch-linux-install-guide.md#210-exit-chroot) step.
