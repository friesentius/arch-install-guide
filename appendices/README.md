# Appendices

These appendices cover optional or specialized setups that build on or replace parts of the
[core install guide](../arch-linux-install-guide.md). Each one says exactly which core-guide
section(s) it replaces or adds to.

| Appendix | What it's for | Relation to the core guide |
| --- | --- | --- |
| [`lvm-disk-layout.md`](lvm-disk-layout.md) | Split root/var/tmp/swap/home into separate LVM logical volumes instead of one root partition and a swapfile. | Replaces the core guide's partition/format/mount/swap steps and adds the `lvm2` initramfs hook. Want LVM instead of a single partition? Use this instead of the core guide's disk-layout sections. |
| [`disk-encryption.md`](disk-encryption.md) | Full-disk encryption with LUKS (with a note on combining it with LVM). | Replaces the core guide's partition/format/mount steps, the swap section, the initramfs `HOOKS` line, and the bootloader `root=`/cmdline value. A **pre-install decision** - read it before 5.0 Partition the Disk, not after. |
| [`limine-bootloader.md`](limine-bootloader.md) | Boot with Limine instead of systemd-boot. | Replaces the core guide's "Install and Configure systemd-boot" step. |
| [`alternate-shell.md`](alternate-shell.md) | Set zsh or fish as your new user's login shell instead of bash. | Adds an optional step in between the core guide's "Add User" and "Configure Privilege Escalation" steps. |
| [`alternate-privilege-escalation.md`](alternate-privilege-escalation.md) | Use `opendoas` instead of `sudo` for privilege escalation. | Replaces the core guide's "Configure Privilege Escalation" step. |
| [`graphics-and-extras.md`](graphics-and-extras.md) | CPU microcode, graphics drivers (with 32-bit/multilib support for things like Steam), and AUR build dependencies. | Adds on top of the core guide once you have a bootable system; nothing here is required to finish the core install. |
| [`firewall.md`](firewall.md) | Enable a firewall (`ufw`) with a default-deny-inbound posture, allowing SSH through if you installed it. | Adds on top of the core guide once you have a bootable system; nothing here is required to finish the core install. |
| [`ssh-hardening.md`](ssh-hardening.md) | Lock down SSH: key-only login, no root login, `fail2ban` against brute-force attempts. | Only relevant if you installed/enabled openssh earlier; adds on top of the core guide's optional `sshd` service. |
| [`automatic-updates.md`](automatic-updates.md) | A scheduled reminder to check for updates, and why fully unattended `pacman -Syu` is risky on Arch specifically. | Adds on top of the core guide once you have a bootable system; nothing here is required. |
| [`system-config.md`](system-config.md) | Alternatives for network management (NetworkManager), power management (`power-profiles-daemon`/TLP), and time sync (`chrony`). | Offers alternatives to the core guide's `iwd`/`dhcpcd` networking and `systemd-timesyncd` steps; nothing here is required. |
| [`shell-config.md`](shell-config.md) | Customizing your shell's config file (`.bashrc`/`.zshrc`/`config.fish`) and managing dotfiles long-term. | Builds on the core guide's Add User step and the [alternate-shell appendix](alternate-shell.md); nothing here is required. |

If you're not sure whether you need an appendix, you probably don't: the core guide on its own
gets you a complete, bootable Arch install.
