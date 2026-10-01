# 6.0 Set Up the Disk: LVM

Separate volumes for root, `/var`, `/tmp`, swap, and `/home`, so a runaway `/var` can't fill
root. Volumes can be resized later.

#### Create the LVM volumes:
```shell
pvcreate /dev/<your-linux-partition>     # e.g. /dev/nvme0n1p2
vgcreate vg /dev/<your-linux-partition>  # e.g. /dev/nvme0n1p2
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
echo "D /tmp 1777 root root 1d" > /mnt/etc/tmpfiles.d/clean-tmp.conf
```
Empties `/tmp` at every boot, and clears files older than a day (`1d`) in between.

#### Save the boot settings for this disk:
```shell
mkdir -p /mnt/etc/mkinitcpio.conf.d /mnt/etc/kernel
cat > /mnt/etc/mkinitcpio.conf.d/disk.conf <<'EOF'
MODULES=(vfat)
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block lvm2 filesystems fsck)
EOF
echo "root=/dev/vg/root rw" > /mnt/etc/kernel/cmdline
cat /mnt/etc/kernel/cmdline
```
`disk.conf` sets what the initramfs loads at boot: FAT32 support for the EFI partition, and the
hooks for this disk layout. The hooks use `udev`, `keymap`, and `consolefont` in place of the
stock `systemd` and `sd-vconsole`, so your console keymap applies early in boot. `lvm2` finds the
volumes before root is mounted. `/etc/kernel/cmdline` holds the kernel boot options the
bootloader will use.

Continue at [7.0 Install Essential Packages](../arch-linux-install-guide.md#70-install-essential-packages).
