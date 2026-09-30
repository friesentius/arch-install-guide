# Branch: Limine Bootloader

Replaces the [main guide](../arch-linux-install-guide.md)'s 20.0 Install and Configure
systemd-boot, for booting with Limine instead. Run it inside the chroot.

## 1.0 Install limine
```shell
pacman -S limine
```

## 2.0 Install limine Bootloader
```shell
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
```
Copies Limine to the EFI partition's fallback boot path, which most UEFI firmware boots
automatically without a registered boot entry.

## 3.0 Create /boot/limine.conf

#### Find your cmdline value
Check your disk layout with `lsblk -f`: look for `crypto_LUKS` and/or `LVM2_member`. If you see
`crypto_LUKS`, get the UUID of the encrypted partition itself (not the `/dev/mapper` device):
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```

**Plain partition layout (default):**
```
cmdline: root=/dev/<your-root-partition> rw rootfstype=ext4 add_efi_memmap vsyscall=none
```

**LVM only:**
```
cmdline: root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none
```

**Encryption only:**
```
cmdline: cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/mapper/cryptroot rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
`cryptdevice=` tells the `encrypt` hook which partition to unlock (by the UUID `blkid` printed)
and to name it `cryptroot`; `root=` then points at the unlocked device.

**LVM + encryption (LVM-on-LUKS):**
```
cmdline: cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Same `cryptdevice=`, but `root=` points at the logical volume the `lvm2` hook finds inside the
unlocked container.

#### Write the config
```shell
nano /boot/limine.conf
```
```conf
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux.img
    cmdline: root=/dev/<your-root-partition> rw rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: root=/dev/<your-root-partition> rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
Replace both `cmdline` lines with yours from above. The fallback entry boots an initramfs with
more drivers, as a recovery option.

## Continue in the main guide
Continue at [21.0 Exit Chroot](../arch-linux-install-guide.md#210-exit-chroot).
