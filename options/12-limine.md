# 12.0 Install the Bootloader: Limine

```shell
pacman -S limine
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
```
Copies Limine to the EFI partition's fallback boot path, which most UEFI firmware boots
automatically without a registered boot entry.

#### Create /boot/limine.conf:
Print your disk's kernel command line, to copy into both entries below:
```shell
cat /etc/kernel/cmdline
```
```shell
nano /boot/limine.conf
```
```conf
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux.img
    cmdline: <your-kernel-cmdline> rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: <your-kernel-cmdline> rootfstype=ext4 add_efi_memmap vsyscall=none
```
Each `cmdline` is filled in from `/etc/kernel/cmdline`, the boot options for your disk. The
fallback entry boots an initramfs with more drivers, as a recovery option.

Continue at [13.0 Set Hostname](../arch-linux-install-guide.md#130-set-hostname).
