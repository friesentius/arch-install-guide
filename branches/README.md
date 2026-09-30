# Branches

Optional routes off the [main guide](../arch-linux-install-guide.md). The main guide links to
each one at the step where the choice is made. A branch contains every step that differs for its
choice, including steps that are otherwise the same as the main guide, and then sends you back to
the first main-guide step after which nothing differs.

| Branch | What it's for | Forks from | Rejoins at |
| --- | --- | --- | --- |
| [`lvm-disk-layout.md`](lvm-disk-layout.md) | Separate LVM volumes for root/var/tmp/swap/home instead of one root partition and a swapfile. | 5.0 Choose Your Disk Layout | 16.0 Network Configuration |
| [`disk-encryption.md`](disk-encryption.md) | Full-disk encryption with LUKS. | 5.0 Choose Your Disk Layout | 16.0 Network Configuration |
| [`lvm-on-luks.md`](lvm-on-luks.md) | LVM volumes inside one LUKS-encrypted partition. | 5.0 Choose Your Disk Layout | 16.0 Network Configuration |
| [`limine-bootloader.md`](limine-bootloader.md) | Limine instead of systemd-boot. | 15.0 Install and Configure systemd-boot | 16.0 Network Configuration |
| [`limine-bootloader-lvm.md`](limine-bootloader-lvm.md) | Limine, for the LVM layout. | 15.0 in `lvm-disk-layout.md` | 16.0 Network Configuration |
| [`limine-bootloader-encrypted.md`](limine-bootloader-encrypted.md) | Limine, for the encrypted layout. | 15.0 in `disk-encryption.md` | 16.0 Network Configuration |
| [`limine-bootloader-lvm-on-luks.md`](limine-bootloader-lvm-on-luks.md) | Limine, for LVM on LUKS. | 15.0 in `lvm-on-luks.md` | 16.0 Network Configuration |
| [`alternate-privilege-escalation.md`](alternate-privilege-escalation.md) | `opendoas` instead of `sudo`. | 20.0 Configure Privilege Escalation | 21.0 Exit Chroot |
| [`graphics-and-extras.md`](graphics-and-extras.md) | CPU microcode, graphics drivers with 32-bit support, and AUR build tools. | 25.0 Graphics, Microcode, and AUR Tools | 26.0 Firewall |
| [`firewall.md`](firewall.md) | `ufw` blocking unsolicited inbound connections. | 26.0 Firewall | 27.0 SSH Server |
| [`ssh-server.md`](ssh-server.md) | OpenSSH with key-only login, no root login, and `fail2ban`. | 27.0 SSH Server | 28.0 Update Hygiene |
| [`automatic-updates.md`](automatic-updates.md) | A daily check for pending updates, and why unattended upgrades are risky on Arch. | 28.0 Update Hygiene | 29.0 System Configuration |
| [`system-config.md`](system-config.md) | NetworkManager, power management (`power-profiles-daemon`/TLP), and `chrony`. | 29.0 System Configuration | 30.0 Shell Configuration |
| [`shell-config.md`](shell-config.md) | Customizing bash's `~/.bashrc`. | 30.0 Shell Configuration | 31.0 Dotfiles |
| [`alternate-shell.md`](alternate-shell.md) | zsh or fish as your login shell, with a starter config. | 30.0 Shell Configuration | 31.0 Dotfiles |
