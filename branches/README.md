# Branches

These are the scenic routes off the [main guide](../arch-linux-install-guide.md)'s path. Each one
is optional, and each one says exactly which step of the main guide it forks from, which steps (if
any) it replaces, and which step you rejoin when you're done - the main guide's own step text
links to these same branches at the point where the choice is made.

Where a branch changes something a later step needs (an initramfs hook, an fstab line, a
bootloader `root=` value), that later main-guide step spells out the branch-specific option
directly, side by side with the default, instead of sending you back here - so once you rejoin,
you never need to revisit a branch file to know what to type.

| Branch | What it's for | Forks from | Rejoins at |
| --- | --- | --- | --- |
| [`lvm-disk-layout.md`](lvm-disk-layout.md) | Split root/var/tmp/swap/home into separate LVM logical volumes instead of one root partition and a swapfile. | 5.0 Choose Your Disk Layout | 9.0 Install Essential Packages, once - steps 9.0, 14.0, and 15.0 each already show the LVM option directly |
| [`disk-encryption.md`](disk-encryption.md) | Full-disk encryption with LUKS (with a note on combining it with LVM). A **pre-install decision** - read it before 5.0, not after. | 5.0 Choose Your Disk Layout | 9.0 Install Essential Packages, once - steps 14.0, 15.0, and 20.0 each already show the encrypted option directly |
| [`limine-bootloader.md`](limine-bootloader.md) | Boot with Limine instead of systemd-boot. | 20.0 Install and Configure systemd-boot | 21.0 Exit Chroot |
| [`alternate-shell.md`](alternate-shell.md) | Set zsh or fish as your new user's login shell instead of bash. | 18.0 Add User (an insert, not a replacement) | 19.0 Configure Privilege Escalation |
| [`alternate-privilege-escalation.md`](alternate-privilege-escalation.md) | Use `opendoas` instead of `sudo` for privilege escalation. | 19.0 Configure Privilege Escalation | 20.0 Install and Configure systemd-boot |
| [`graphics-and-extras.md`](graphics-and-extras.md) | CPU microcode, graphics drivers (with 32-bit/multilib support for things like Steam), and AUR build dependencies. | 25.0 Graphics, Microcode, and AUR Tools | 26.0 Firewall |
| [`firewall.md`](firewall.md) | Enable a firewall (`ufw`) with a default-deny-inbound posture, allowing SSH through if you installed it. | 26.0 Firewall | 27.0 SSH Hardening |
| [`ssh-hardening.md`](ssh-hardening.md) | Lock down SSH: key-only login, no root login, `fail2ban` against brute-force attempts. Only relevant if you installed/enabled openssh earlier. | 27.0 SSH Hardening | 28.0 Update Hygiene |
| [`automatic-updates.md`](automatic-updates.md) | A scheduled reminder to check for updates, and why fully unattended `pacman -Syu` is risky on Arch specifically. | 28.0 Update Hygiene | 29.0 System Configuration |
| [`system-config.md`](system-config.md) | Alternatives for network management (NetworkManager), power management (`power-profiles-daemon`/TLP), and time sync (`chrony`). | 29.0 System Configuration | 30.0 Shell Configuration and Dotfiles |
| [`shell-config.md`](shell-config.md) | Customizing your shell's config file (`.bashrc`/`.zshrc`/`config.fish`) and managing dotfiles long-term. Pairs with `alternate-shell.md`. | 30.0 Shell Configuration and Dotfiles (the last step) | n/a - end of the path |

If you're not sure whether you need a branch, you probably don't: taking the default at every
step gets you a complete, bootable, working Arch install. Branches that fork from the same step
(none currently do except disk layout's two, which are meant to combine) can be combined by
reading both - `disk-encryption.md` explains combining with `lvm-disk-layout.md` directly.
