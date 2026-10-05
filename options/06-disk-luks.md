# 6.0 Set Up the Disk: Encrypted (LUKS)

Encrypts the Linux partition, so the disk is unreadable without your passphrase if the machine is
lost, stolen, or accessed while powered off.

#### Encrypt the partition:
```shell
cryptsetup luksFormat /dev/<your-linux-partition>      # e.g. /dev/nvme0n1p2
cryptsetup open /dev/<your-linux-partition> cryptroot  # e.g. /dev/nvme0n1p2
```
Type `YES` to confirm, then set a passphrase. **There is no recovery if you forget it**, so store
it somewhere safe, such as a password manager. `open` unlocks the partition as
`/dev/mapper/cryptroot`. The EFI partition stays unencrypted, because UEFI firmware must read it.

#### Format and mount:
```shell
mkfs.fat -F32 /dev/<your-efi-partition>    # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/cryptroot
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
UEFI requires FAT32 on the EFI partition; the unlocked partition gets ext4. The new system gets
installed under `/mnt`.

#### Create swap:
```shell
dd if=/dev/zero of=/mnt/swapfile bs=1M count=4096 status=progress
chmod 600 /mnt/swapfile
mkswap /mnt/swapfile
swapon /mnt/swapfile
```
A 4G swapfile (`count=4096` MiB; raise it to roughly your RAM size if you want hibernation). It's
active now, so 8.0 Generate fstab records it automatically. The swapfile sits inside the
encrypted root, so it is encrypted too.

#### Create /mnt/etc/mkinitcpio.conf.d/disk.conf:
```shell
mkdir -p /mnt/etc/mkinitcpio.conf.d /mnt/etc/kernel
nano /mnt/etc/mkinitcpio.conf.d/disk.conf
```
Write the following into the file:
```conf
# /mnt/etc/mkinitcpio.conf.d/disk.conf
MODULES=(vfat)
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```
Sets what the initramfs loads at boot: FAT32 support for the EFI partition, and the hooks for
this disk layout. The hooks use `udev`, `keymap`, and `consolefont` in place of the stock
`systemd` and `sd-vconsole`, so your console keymap applies early in boot. `encrypt` asks for your
passphrase and unlocks the disk before root is mounted.

#### Create /mnt/etc/kernel/cmdline:
Look up the Linux partition's UUID:
```shell
blkid -s UUID -o value /dev/<your-linux-partition>  # e.g. /dev/nvme0n1p2
```
```shell
nano /mnt/etc/kernel/cmdline
```
Write the following into the file:
```conf
cryptdevice=UUID=<your-linux-partition-uuid>:cryptroot root=/dev/mapper/cryptroot rw
```
Holds the kernel boot options the bootloader will use. `cryptdevice=` names the partition to
unlock (by UUID, since device names can change) and calls it `cryptroot`; `root=` points at the
unlocked filesystem.

Continue at [7.0 Install Essential Packages](../arch-linux-install-guide.md#70-install-essential-packages).
