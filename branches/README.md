# Branches

Optional routes off the [main guide](../arch-linux-install-guide.md). The main guide links to
each one at the step where the choice is made, and each ends with a link back to where you
rejoin. When a branch changes a later step (an initramfs hook, a boot option), that later step
lists every variant itself, so you never need to come back here.

| Branch | What it's for | Forks from | Rejoins at |
| --- | --- | --- | --- |
| [`lvm-disk-layout.md`](lvm-disk-layout.md) | Separate LVM volumes for root/var/tmp/swap/home instead of one root partition and a swapfile. | 5.0 Choose Your Disk Layout | 9.0 Install Essential Packages |
| [`disk-encryption.md`](disk-encryption.md) | Full-disk encryption with LUKS, optionally combined with LVM. | 5.0 Choose Your Disk Layout | 9.0 Install Essential Packages |
| [`alternate-shell.md`](alternate-shell.md) | zsh or fish as your login shell instead of bash. | 18.0 Add User (an addition, not a replacement) | 19.0 Configure Privilege Escalation |
| [`alternate-privilege-escalation.md`](alternate-privilege-escalation.md) | `opendoas` instead of `sudo`. | 19.0 Configure Privilege Escalation | 20.0 Install and Configure systemd-boot |
| [`limine-bootloader.md`](limine-bootloader.md) | Limine instead of systemd-boot. | 20.0 Install and Configure systemd-boot | 21.0 Exit Chroot |
| [`graphics-and-extras.md`](graphics-and-extras.md) | CPU microcode, graphics drivers with 32-bit support, and AUR build tools. | 25.0 Graphics, Microcode, and AUR Tools | 26.0 Firewall |
| [`firewall.md`](firewall.md) | `ufw` blocking unsolicited inbound connections. | 26.0 Firewall | 27.0 SSH Hardening |
| [`ssh-hardening.md`](ssh-hardening.md) | Key-only SSH login, no root login, and `fail2ban`. | 27.0 SSH Hardening | 28.0 Update Hygiene |
| [`automatic-updates.md`](automatic-updates.md) | A daily check for pending updates, and why unattended upgrades are risky on Arch. | 28.0 Update Hygiene | 29.0 System Configuration |
| [`system-config.md`](system-config.md) | NetworkManager, power management (`power-profiles-daemon`/TLP), and `chrony`. | 29.0 System Configuration | 30.0 Shell Configuration and Dotfiles |
| [`shell-config.md`](shell-config.md) | Shell config files and dotfiles management. | 30.0 Shell Configuration and Dotfiles | n/a - end of the path |

The two disk-layout branches combine: `disk-encryption.md` explains running it with
`lvm-disk-layout.md` for LVM-on-LUKS.
