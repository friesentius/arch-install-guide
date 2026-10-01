# 12.0 Install the Bootloader: systemd-boot (default)

```shell
bootctl install
cat > /boot/loader/loader.conf <<EOF
default arch.conf
timeout 3
console-mode max
editor no
EOF
cat > /boot/loader/entries/arch.conf <<EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options $(cat /etc/kernel/cmdline)
EOF
cat > /boot/loader/entries/arch-fallback.conf <<EOF
title   Arch Linux (fallback initramfs)
linux   /vmlinuz-linux
initrd  /initramfs-linux-fallback.img
options $(cat /etc/kernel/cmdline)
EOF
cat /boot/loader/entries/arch.conf
```
`bootctl install` copies systemd-boot onto the EFI partition and registers it with the UEFI
firmware. `loader.conf` boots `arch.conf` after 3 seconds, uses the highest text resolution, and
disables editing kernel parameters from the boot menu, so someone at the keyboard can't change
them. Each entry's `options` line is filled in from `/etc/kernel/cmdline`, the boot options for
your disk. The fallback entry boots an initramfs with more drivers, as a recovery option.

Continue at [13.0 Set Hostname](../arch-linux-install-guide.md#130-set-hostname).
