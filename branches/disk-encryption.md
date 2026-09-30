# Branch: Full-Disk Encryption (LUKS)

Forks from the [main guide](../arch-linux-install-guide.md)'s
[5.0 Choose Your Disk Layout](../arch-linux-install-guide.md#50-choose-your-disk-layout) and
replaces steps 6.0-8.0. It encrypts the root partition with LUKS via `cryptsetup`, so the disk is
unreadable without your passphrase if the machine is lost, stolen, or accessed while powered off.
This must be decided before partitioning; it can't be added to an installed system.

The EFI partition stays unencrypted, because UEFI firmware must read it. `cryptsetup` is already
pulled in by `base`, so `pacstrap` needs nothing extra.

**Want LVM volumes too (LVM-on-LUKS)?** Run 1.0-3.0 below to get an unlocked
`/dev/mapper/cryptroot`, then skip this branch's 4.0-5.0 and run the
[LVM branch](lvm-disk-layout.md)'s 2.0-5.0 with `/dev/mapper/cryptroot` in place of
`/dev/<your-lvm-partition>`. That branch's end sends you back to the main guide. (The reverse,
encrypting individual logical volumes, needs a separate unlock per volume and isn't covered.)

## 1.0 Partition the Disk
```shell
cfdisk /dev/<your-disk>  # e.g. /dev/nvme0n1
```
Create an EFI System partition and one Linux partition to hold the LUKS container:

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
Run `lsblk` for the new partition names (`/dev/<your-efi-partition>` and
`/dev/<your-root-partition>` below). NVMe disks add a `p` before the number; SATA/virtio disks
don't.

## 2.0 Create the LUKS Container
```shell
cryptsetup luksFormat /dev/<your-root-partition>  # e.g. /dev/nvme0n1p2
```
Type `YES` to confirm, then set a passphrase. **There is no recovery if you forget it**, so store
it somewhere safe, such as a password manager.

## 3.0 Open the LUKS Container
```shell
cryptsetup open /dev/<your-root-partition> cryptroot  # e.g. /dev/nvme0n1p2
```
Unlocks the container as `/dev/mapper/cryptroot`. `cryptroot` is a name you choose here; the
main guide's boot entry uses the same name. From now on, format and mount
`/dev/mapper/cryptroot`, not the raw partition.

## 4.0 Format the Partitions
```shell
mkfs.fat -F32 /dev/<your-efi-partition>  # e.g. /dev/nvme0n1p1
mkfs.ext4 /dev/mapper/cryptroot
```

## 5.0 Mount the Partitions
```shell
mount /dev/mapper/cryptroot /mnt
mkdir /mnt/boot
mount /dev/<your-efi-partition> /mnt/boot  # e.g. /dev/nvme0n1p1
```

## Continue in the main guide
Continue at [9.0 Install Essential Packages](../arch-linux-install-guide.md#90-install-essential-packages).
Steps 9.0, 14.0, 15.0, and 20.0 each list a variant per disk layout, with an `lsblk -f` check
that tells you which one is yours.
