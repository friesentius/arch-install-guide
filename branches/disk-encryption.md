# Branch: Full-Disk Encryption (LUKS)

This branch is different from the others: it's not a post-install add-on, it's a **pre-install
decision**. Whether to encrypt has to be decided before you partition, because it changes the
partitioning, formatting, and mounting steps you'd otherwise do in the main guide - by the time
you reach
[24.0 Verify Installation](../arch-linux-install-guide.md#240-verify-installation) it's too late
to retrofit without redoing those steps. If you want encryption, read this branch *before* you
reach the main guide's
[5.0 Choose Your Disk Layout](../arch-linux-install-guide.md#50-choose-your-disk-layout).

This branch replaces the main guide's 6.0-8.0 (Partition the Disk / Format the Partitions /
Mount the Partitions) outright, below. Three later main-guide steps each already lay out an
encrypted-setup option directly, side by side with the default - you don't need to come back
here for them, just pick that line when you reach each one: 14.0 Configure Swap (a swapfile
inside encrypted root needs no different commands at all), 15.0 Initramfs Configuration (adds
the `encrypt` hook), and 20.0 Install and Configure systemd-boot (uses a `cryptdevice=` boot
option, with the UUID lookup spelled out right there). Everything else in the main guide -
keyboard layout, network setup, pacstrap, fstab, chroot, locale, hostname, networking services,
root password, user creation, and privilege escalation - continues unchanged around this branch.

**Use this branch if:** you want the contents of your disk unreadable to anyone without your
passphrase if the machine is lost, stolen, or otherwise physically accessed while powered off -
a laptop is the classic case. It uses LUKS (Linux Unified Key Setup), the standard Linux
disk-encryption format, via `cryptsetup`.

**What stays unencrypted:** the EFI system partition, same as every layout in this guide - UEFI
firmware needs to read it directly, so it can't be encrypted. Everything past that (root, and
everything you'd otherwise put under it) is what this branch encrypts.

**Also needed:** nothing extra to `pacstrap` - unlike the LVM branch's `lvm2`, `cryptsetup` is
already part of the `base` package group the main guide installs.

**Combining with LVM (LVM-on-LUKS):** if you also want LVM's separate root/var/tmp/swap/home
volumes, run this branch's 1.0-3.0 below (partition, then `cryptsetup luksFormat`, then
`cryptsetup open`) to get an opened `/dev/mapper/cryptroot` device. Then switch to the
[LVM branch](lvm-disk-layout.md) and run its 2.0 Create LVM Physical Volume & Volume Group
through 5.0 Mount Filesystems, using `/dev/mapper/cryptroot` in place of `/dev/<your-lvm-partition>`
everywhere that branch says to `pvcreate`/`vgcreate` against it - skip this branch's own 4.0
Format the Partitions and 5.0 Mount the Partitions below, since the LVM branch's steps format
and mount the logical volumes instead. From there, continue exactly as either branch alone
describes: main guide steps 9.0, 14.0, 15.0, and 20.0 each include the LVM+encryption
combination as one of their listed options. The reverse, LUKS-on-LVM (encrypting individual
logical volumes instead of the one underlying partition), is also a real setup, but needs a
separate `cryptsetup open` per volume at boot and isn't covered here.

## 1.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Same as the main guide: an EFI System partition, plus one Linux partition - except here the
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
As in the main guide, run `lsblk` after writing to find your actual partition device names
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
rest of this branch, and by the `cryptdevice=...:cryptroot` boot option you'll set at main guide
step 20.0), not something to look up. Everything from here on (formatting, mounting) targets
`/dev/mapper/cryptroot`, not the raw partition directly.

## 4.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/cryptroot
```
Same filesystems as the main guide (FAT32 on the EFI partition, ext4 on root) - just applied to
the decrypted mapper device for root, instead of the raw partition.

## 5.0 Mount the Partitions
```shell
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```
Same as the main guide's mount step, with `/dev/mapper/cryptroot` in place of the raw root
partition. `/boot` remains unencrypted, same as always, since UEFI needs to read it directly.

## Continue in the main guide
Your disk layout is done. Jump to
[9.0 Install Essential Packages](../arch-linux-install-guide.md#90-install-essential-packages)
and follow the main guide's numbered steps normally from there - 14.0 Configure Swap, 15.0
Initramfs Configuration, and 20.0 Install and Configure systemd-boot each lay out the encrypted
option you need directly, right next to the default, so just take that option at each.
