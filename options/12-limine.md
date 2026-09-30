# 12.0 Install the Bootloader: Limine

```shell
pacman -S limine
mkdir -p /boot/EFI/BOOT
cp /usr/share/limine/BOOTX64.EFI /boot/EFI/BOOT/
cat > /boot/limine.conf <<EOF
timeout: 5

/Arch Linux (linux)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux.img
    cmdline: $(cat /etc/kernel/cmdline) rootfstype=ext4 add_efi_memmap vsyscall=none

/Arch Linux (linux-fallback)
    protocol: linux
    path: boot():/vmlinuz-linux
    module_path: boot():/initramfs-linux-fallback.img
    cmdline: $(cat /etc/kernel/cmdline) rootfstype=ext4 add_efi_memmap vsyscall=none
EOF
cat /boot/limine.conf
```
Copies Limine to the EFI partition's fallback boot path, which most UEFI firmware boots
automatically without a registered boot entry, and writes its menu. Each `cmdline` is filled in
from `/etc/kernel/cmdline`, the boot options for your disk. The fallback entry boots an initramfs
with more drivers, as a recovery option.

Continue at [13.0 Set Hostname](../arch-linux-install-guide.md#130-set-hostname).
