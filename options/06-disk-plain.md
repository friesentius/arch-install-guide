# 6.0 Set Up the Disk: Plain Partitions (default)

One ext4 root filesystem directly on the Linux partition, plus a swapfile.

#### Format and mount:
```shell
mkfs.fat -F32 /dev/<your-efi-partition>    # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/<your-linux-partition>      # e.g. /dev/nvme0n1p2
mount /dev/<your-linux-partition> /mnt     # e.g. /dev/nvme0n1p2
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
UEFI requires FAT32 on the EFI partition; root gets ext4. The new system gets installed under
`/mnt`.

#### Create swap:
```shell
dd if=/dev/zero of=/mnt/swapfile bs=1M count=4096 status=progress
chmod 600 /mnt/swapfile
mkswap /mnt/swapfile
swapon /mnt/swapfile
```
A 4G swapfile (`count=4096` MiB; raise it to roughly your RAM size if you want hibernation). It's
active now, so 8.0 Generate fstab records it automatically.

#### Save the boot settings for this disk:
```shell
mkdir -p /mnt/etc/mkinitcpio.conf.d /mnt/etc/kernel
cat > /mnt/etc/mkinitcpio.conf.d/disk.conf <<'EOF'
MODULES=(vfat)
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block filesystems fsck)
EOF
echo "root=UUID=$(blkid -s UUID -o value /dev/<your-linux-partition>) rw" > /mnt/etc/kernel/cmdline
cat /mnt/etc/kernel/cmdline
```
`disk.conf` sets what the initramfs loads at boot: FAT32 support for the EFI partition, and the
hooks for this disk layout. The hooks use `udev`, `keymap`, and `consolefont` in place of the
stock `systemd` and `sd-vconsole`, so your console keymap applies early in boot.
`/etc/kernel/cmdline` holds the kernel boot options the bootloader will use.

Continue at [7.0 Install Essential Packages](../arch-linux-install-guide.md#70-install-essential-packages).
