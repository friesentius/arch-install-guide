# 6.0 Set Up the Disk: LVM inside LUKS

Separate volumes for root, `/var`, `/tmp`, swap, and `/home`, all inside one encrypted partition:
one passphrase unlocks them all, and a runaway `/var` can't fill root.

#### Encrypt the partition:
```shell
cryptsetup luksFormat /dev/<your-linux-partition>      # e.g. /dev/nvme0n1p2
cryptsetup open /dev/<your-linux-partition> cryptroot  # e.g. /dev/nvme0n1p2
```
Type `YES` to confirm, then set a passphrase. **There is no recovery if you forget it**, so store
it somewhere safe, such as a password manager. `open` unlocks the partition as
`/dev/mapper/cryptroot`. The EFI partition stays unencrypted, because UEFI firmware must read it.

#### Create the LVM volumes:
```shell
pvcreate /dev/mapper/cryptroot
vgcreate vg /dev/mapper/cryptroot
lvcreate -L 20G vg -n root               # OS and packages
lvcreate -L 20G vg -n var                # logs, caches, databases
lvcreate -L 8G vg -n tmp                 # temporary files
lvcreate -L 4G vg -n swap                # adjust to match RAM if hibernating
lvcreate -l 100%FREE vg -n home          # remaining space
```
Creates a volume group named `vg` and carves the volumes out of it. Adjust the sizes to your
drive; on a 256G drive, for example, 10G `/var` and 4G `/tmp` leave more for `/home`.

#### Format and mount:
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 -L "Arch Root" /dev/vg/root
mkfs.ext4 -L "Arch Var"  /dev/vg/var
mkfs.ext4 -L "Arch Tmp"  /dev/vg/tmp
mkfs.ext4 -L "Arch Home" /dev/vg/home
mount /dev/vg/root /mnt
mkdir -p /mnt/{home,var,tmp,boot}
mount /dev/vg/home /mnt/home
mount /dev/vg/var  /mnt/var
mount -o noatime,nosuid,nodev /dev/vg/tmp /mnt/tmp
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
UEFI requires FAT32 on the EFI partition; the volumes get ext4. `/tmp` is mounted with `nosuid`
and `nodev`, which stop setuid binaries and device files from working in world-writable `/tmp`.

#### Create swap:
```shell
mkswap /dev/vg/swap
swapon /dev/vg/swap
```
The swap volume is active now, so 8.0 Generate fstab records it automatically.

#### Optional: empty /tmp at every boot:
```shell
mkdir -p /mnt/etc/tmpfiles.d
nano /mnt/etc/tmpfiles.d/clean-tmp.conf
```
```conf
D /tmp 1777 root root 1d
```
Empties `/tmp` at every boot, and clears files older than a day (`1d`) in between.

#### Create /mnt/etc/mkinitcpio.conf.d/disk.conf:
```shell
mkdir -p /mnt/etc/mkinitcpio.conf.d /mnt/etc/kernel
nano /mnt/etc/mkinitcpio.conf.d/disk.conf
```
```conf
MODULES=(vfat)
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt lvm2 filesystems fsck)
```
Sets what the initramfs loads at boot: FAT32 support for the EFI partition, and the hooks for
this disk layout. The hooks use `udev`, `keymap`, and `consolefont` in place of the stock
`systemd` and `sd-vconsole`, so your console keymap applies early in boot. `encrypt` asks for your
passphrase and unlocks the disk, then `lvm2` finds the volumes inside it, before root is mounted.

#### Create /mnt/etc/kernel/cmdline:
Look up the Linux partition's UUID:
```shell
blkid -s UUID -o value /dev/<your-linux-partition>  # e.g. /dev/nvme0n1p2
```
```shell
nano /mnt/etc/kernel/cmdline
```
```conf
cryptdevice=UUID=<your-linux-partition-uuid>:cryptroot root=/dev/vg/root rw
```
Holds the kernel boot options the bootloader will use. `cryptdevice=` names the partition to
unlock (by UUID, since device names can change) and calls it `cryptroot`; `root=` points at the
root volume inside it.

Continue at [7.0 Install Essential Packages](../arch-linux-install-guide.md#70-install-essential-packages).
