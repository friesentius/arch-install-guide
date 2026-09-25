# Appendices

These appendices cover optional or specialized setups that build on or replace parts of the
[core install guide](../arch-linux-install-guide.md). Each one says exactly which core-guide
section(s) it replaces or adds to.

| Appendix | What it's for | Relation to the core guide |
| --- | --- | --- |
| [`lvm-disk-layout.md`](lvm-disk-layout.md) | Split root/var/tmp/swap/home into separate LVM logical volumes instead of one root partition and a swapfile. | Replaces the core guide's partition/format/mount/swap steps and adds the `lvm2` initramfs hook. Want LVM instead of a single partition? Use this instead of the core guide's disk-layout sections. |
| [`limine-bootloader.md`](limine-bootloader.md) | Boot with Limine instead of systemd-boot. | Replaces the core guide's "Install and Configure systemd-boot" step. |
| [`graphics-and-extras.md`](graphics-and-extras.md) | CPU microcode, graphics drivers (with 32-bit/multilib support for things like Steam), and AUR build dependencies. | Adds on top of the core guide once you have a bootable system; nothing here is required to finish the core install. |

If you're not sure whether you need an appendix, you probably don't: the core guide on its own
gets you a complete, bootable Arch install.
