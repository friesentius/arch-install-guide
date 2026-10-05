# 29.0 Update Hygiene: Daily Update Reminder

A daily list of pending updates, without installing them.

**Why not run `pacman -Syu` on a timer?** Some Arch upgrades need manual steps (a changed config
format, a renamed package, a migration), announced on
[archlinux.org/news](https://archlinux.org/news/) before they land. An unattended upgrade skips
reading them, which can leave the system half-upgraded or unbootable. So this only automates the
reminder.

#### Install checkupdates:
```shell
sudo pacman -S pacman-contrib
```
Provides `checkupdates`, which lists pending updates using a temporary copy of the package
database. Unlike `pacman -Sy`, it leaves your real database untouched, so it's safe to run on a
schedule.

#### Create /etc/systemd/system/checkupdates.service:
```shell
sudo nano /etc/systemd/system/checkupdates.service
```
```conf
# /etc/systemd/system/checkupdates.service
[Unit]
Description=Check for pending pacman updates

[Service]
Type=oneshot
ExecStart=-/usr/bin/checkupdates
StandardOutput=journal
```

#### Create /etc/systemd/system/checkupdates.timer:
```shell
sudo nano /etc/systemd/system/checkupdates.timer
```
```conf
# /etc/systemd/system/checkupdates.timer
[Unit]
Description=Run checkupdates daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```
```shell
sudo systemctl enable --now checkupdates.timer
```
The service logs the list of pending updates to the journal and never installs anything; the
leading `-` stops systemd from counting `checkupdates`' "no updates" exit code as a failure. The
timer runs it daily, and `Persistent=true` runs a missed check at next boot.

#### See what it found:
```shell
journalctl -u checkupdates.service
```
Your cue to read the news and run `sudo pacman -Syu`.

#### Optional: make pacman enforce reading the news
`informant` (from the AUR) blocks `pacman -Syu` while there's an unread news entry, until you
mark it read:
```shell
sudo pacman -S --needed binutils make gcc pkg-config fakeroot debugedit git
git clone https://aur.archlinux.org/informant.git
cd informant && makepkg -si
```
The first line installs the AUR build tools (`--needed` skips any already installed); `makepkg
-si` builds the package and installs it with its dependencies.

Continue at [30.0 Network Manager](../arch-linux-install-guide.md#300-network-manager).
