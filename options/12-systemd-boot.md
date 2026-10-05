# 12.0 Install the Bootloader: systemd-boot (default)

```shell
bootctl install
```
Copies systemd-boot onto the EFI partition and registers it with the UEFI firmware.

#### Create /boot/loader/loader.conf:
```shell
nano /boot/loader/loader.conf
```
Write the following into the file:
```conf
# /boot/loader/loader.conf
default arch.conf
timeout 3
console-mode max
editor no
```
Boots `arch.conf` after 3 seconds, uses the highest text resolution, and disables editing kernel
parameters from the boot menu, so someone at the keyboard can't change them.

#### Create the boot entries:
Print your disk's kernel command line, to copy into both entries below:
```shell
cat /etc/kernel/cmdline
```
```shell
nano /boot/loader/entries/arch.conf
```
Write the following into the file:
```conf
# /boot/loader/entries/arch.conf
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options <your-kernel-cmdline>
```
```shell
nano /boot/loader/entries/arch-fallback.conf
```
Write the following into the file:
```conf
# /boot/loader/entries/arch-fallback.conf
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options <your-kernel-cmdline>
```
Each entry's `options` line is filled in from `/etc/kernel/cmdline`, the boot options for your
disk. The fallback entry boots an initramfs with more drivers, as a recovery option.

Continue at [13.0 Set Hostname](../arch-linux-install-guide.md#130-set-hostname).
