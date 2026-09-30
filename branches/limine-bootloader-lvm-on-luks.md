# Branch: Limine Bootloader (LVM on LUKS)

Replaces the 15.0 Install and Configure systemd-boot step of the [LVM on LUKS
branch](lvm-on-luks.md), for booting with Limine instead. Run it inside the chroot.

## 15.0 Install and Configure Limine
```shell
pacman -S limine
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
```
Copies Limine to the EFI partition's fallback boot path, which most UEFI firmware boots
automatically without a registered boot entry.

#### Find your root partition's UUID:
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```
Use the UUID of the encrypted partition itself, not of `/dev/mapper/cryptroot`.

#### Create /boot/limine.conf:
```shell
nano /boot/limine.conf
```
```conf
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux.img
    cmdline: cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/vg/root rw rootfstype=ext4 add_efi_memmap vsyscall=none
```
`cryptdevice=` tells the `encrypt` hook which partition to unlock (replace
`<your-root-partition-uuid>` with the UUID `blkid` printed) and to name it `cryptroot`; `root=`
then points at the root logical volume inside it. The fallback entry boots an initramfs with more
drivers, as a recovery option.

## Continue in the main guide
Continue at [16.0 Network Configuration](../arch-linux-install-guide.md#160-network-configuration).
