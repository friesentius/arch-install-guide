# Appendix: Full-Disk Encryption (LUKS)

This appendix is different from the others: it's not a post-install add-on, it's a **pre-install
decision**. Whether to encrypt has to be decided before you partition, because it changes the
partitioning, formatting, mounting, initramfs, and bootloader-cmdline steps you'd otherwise do in
the core guide - by the time you reach
[Verify Installation](../arch-linux-install-guide.md#verify-installation) it's too late to
retrofit without redoing those steps. If you want encryption, read this appendix *before* you
start the core guide's
[5.0 Partition the Disk](../arch-linux-install-guide.md#50-partition-the-disk).

This appendix replaces the core guide's sections 5.0-7.0 (Partition the Disk / Format the
Partitions / Mount the Partitions), the 5.0 Configure Swap section, the `HOOKS` line of 6.0
Initramfs Configuration, and the `root=`/cmdline details of 11.0 Install and Configure
systemd-boot. Everything else in the core guide - keyboard layout, network setup, pacstrap,
fstab, chroot, locale, hostname, networking services, root password, user creation, and
privilege escalation - continues unchanged around this appendix.

**Use this appendix if:** you want the contents of your disk unreadable to anyone without your
passphrase if the machine is lost, stolen, or otherwise physically accessed while powered off -
a laptop is the classic case. It uses LUKS (Linux Unified Key Setup), the standard Linux
disk-encryption format, via `cryptsetup`.

**What stays unencrypted:** the EFI system partition, same as every layout in this guide - UEFI
firmware needs to read it directly, so it can't be encrypted. Everything past that (root, and
everything you'd otherwise put under it) is what this appendix encrypts.

**Also needed:** nothing extra to `pacstrap` - unlike the LVM appendix's `lvm2`, `cryptsetup` is
already part of the `base` package group the core guide installs.

**Combining with LVM:** on its own, this appendix replaces the core guide's plain-partition
layout with a single encrypted root partition (LUKS directly on the partition, keeping the core
guide's simplicity). If you also want LVM's separate root/var/tmp/swap/home volumes, the common
combination is **LVM-on-LUKS**: encrypt the partition first (this appendix's 2.0-3.0 below),
then run the [LVM appendix](lvm-disk-layout.md)'s `pvcreate`/`vgcreate`/`lvcreate` steps against
the opened `/dev/mapper/cryptroot` device instead of a raw partition, and add both the `encrypt`
and `lvm2` initramfs hooks in that order (see 7.0 below). The reverse, LUKS-on-LVM (encrypting
individual logical volumes instead of the one underlying partition), is also a real setup, but
needs a separate `cryptsetup open` per volume at boot and isn't covered here.

## 1.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Same as the core guide: an EFI System partition, plus one Linux partition - except here the
second partition becomes a LUKS container instead of being formatted directly.

```shell
# delete existing partition(s) to make room for your new partition scheme
select [ Delete ]

# Set up the EFI system partition
select [ New ]

Partition Size: 1G

select [ Type ] "EFI System"

# Set up the partition that will become the LUKS container
select [ New ]

Partition Size: accept default value (uses the remaining free space)

select [ Write ]
# example cfdisk output - your sizes and disk name will differ
|Number | Start (sector) | End (sector) | Size   | Code | Name             |
|------ | -------------- | ------------ | ------ | ---- | ---------------- |
|1      | 2048           | 1130495      | 1G     | EF00 | EFI System       |
|2      | 1130496        | 976773134    | 475.9G | 8300 | Linux Filesystem |
```
As in the core guide, run `lsblk` after writing to find your actual partition device names
(`/dev/<your-efi-partition>` and `/dev/<your-root-partition>` below).

## 2.0 Create the LUKS Container
```shell
cryptsetup luksFormat /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```
`luksFormat` initializes the partition as a LUKS container: it prompts you to confirm (type
`YES` in all caps) and then set a passphrase. **This passphrase is everything** - there's no
recovery if you forget it and don't have a key file or backup. Pick something you can remember
or store securely (a password manager, a printed copy in a safe), not something written on a
sticky note next to the machine.

## 3.0 Open the LUKS Container
```shell
cryptsetup open /dev/<your-root-partition> cryptroot  # e.g. /dev/nvme0n1p2
```
Prompts for the passphrase you just set, then exposes the decrypted container as
`/dev/mapper/cryptroot` - `cryptroot` here is a name *you* choose (used consistently for the
rest of this appendix), not something to look up. Everything from here on (formatting,
mounting) targets `/dev/mapper/cryptroot`, not the raw partition directly.

## 4.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/cryptroot
```
Same filesystems as the core guide (FAT32 on the EFI partition, ext4 on root) - just applied to
the decrypted mapper device for root, instead of the raw partition.

## 5.0 Mount the Partitions
```shell
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
Same as the core guide's mount step, with `/dev/mapper/cryptroot` in place of the raw root
partition. `/boot` remains unencrypted, same as always, since UEFI needs to read it directly.

**Continue in the core guide:** your disk layout is done. Jump to
[Base Installation](../arch-linux-install-guide.md#base-installation) and follow the core guide
normally through fstab, chroot, locale, and hostname setup. Come back here when you reach
[5.0 Configure Swap](../arch-linux-install-guide.md#50-configure-swap-swapfile) in Configure the
System - use 6.0 below instead of that step.

## 6.0 Configure Swap (swapfile, inside the encrypted root)
```shell
dd if=/dev/zero of=/swapfile bs=1M count=4096 status=progress
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap defaults 0 0' >> /etc/fstab
```
Identical to the core guide's swapfile steps - since `/swapfile` lives inside the already
encrypted root filesystem, it's automatically covered by the same encryption with no extra work.
This is one advantage of a swapfile-in-root over a separate swap partition/volume here: a
separate, unencrypted swap device would leak decrypted memory contents to disk in the clear,
which a swapfile inside encrypted root doesn't.

## 7.0 Initramfs Configuration: Add the encrypt Hook
```shell
nano /etc/mkinitcpio.conf
```
```conf
HOOKS=(base udev autodetect microcode modconf kms keyboard keymap consolefont block encrypt filesystems fsck)
```
The `encrypt` hook adds the code that prompts for your LUKS passphrase and unlocks the
container at boot, before the `filesystems` hook tries to mount root - it must come after
`block` (which sets up the underlying block devices) and before `filesystems`. (If you're
combining this with the [LVM appendix](lvm-disk-layout.md), add `lvm2` right after `encrypt`, in
the same order: `... block encrypt lvm2 filesystems fsck` - LVM needs the container unlocked
before it can find the volume group inside it.)

```shell
mkinitcpio -P
```

## 8.0 Bootloader cmdline: Reference the Encrypted Device
When you reach the core guide's 11.0 Install and Configure systemd-boot step, first find your
root partition's UUID (the underlying encrypted partition's UUID, not the mapper device's):
```shell
blkid /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```
Then use it in `/boot/loader/entries/arch.conf` (and the fallback entry, if you created one) in
place of the core guide's plain `root=/dev/<your-root-partition>`:
```conf
options cryptdevice=UUID=<your-root-partition-uuid>:cryptroot root=/dev/mapper/cryptroot rw
```
`cryptdevice=UUID=...:cryptroot` tells the `encrypt` hook which device to unlock (by UUID,
since raw device names can shift) and what to name the resulting mapper device (`cryptroot`,
matching what you opened it as back in 3.0); `root=/dev/mapper/cryptroot` then points the kernel
at the now-unlocked device. If you're following [Limine](limine-bootloader.md) instead of
systemd-boot, add the same `cryptdevice=...` text to that appendix's `cmdline` line, in place of
its `root=...` value.

## Continue in the core guide
You've now covered the core guide's disk layout, swap, initramfs `HOOKS`, and bootloader
`root=` steps with their encrypted equivalents. Skip the core guide's own 5.0 Configure Swap and
the `HOOKS`/`root=` details of 6.0/11.0 (you've just done all three above), and pick back up
wherever you left off - typically
[7.0 Enable Networking Services](../arch-linux-install-guide.md#70-enable-networking-services)
if you came here from Configure the System, or straight through to
[Finalize and Reboot](../arch-linux-install-guide.md#finalize-and-reboot) once the bootloader
step above is done.
