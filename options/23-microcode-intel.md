# 23.0 CPU Microcode: Intel

```shell
sudo pacman -S intel-ucode
sudo mkinitcpio -P
```
Rebuilding the initramfs embeds the microcode update, so it loads early at boot with no
bootloader changes.

Continue at [24.0 Graphics Drivers](../arch-linux-install-guide.md#240-graphics-drivers).
