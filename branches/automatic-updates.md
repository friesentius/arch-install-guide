# Branch: Update Hygiene (Scheduled Checks, Not Unattended Upgrades)

The [main guide](../arch-linux-install-guide.md)'s 28.0 Update Hygiene: a daily reminder of
pending updates, without installing them automatically.

**Why not run `pacman -Syu` on a timer?** Some Arch upgrades need manual steps (a changed config
format, a renamed package, a migration). Arch announces these on
[archlinux.org/news](https://archlinux.org/news/) before they land, and an unattended upgrade
skips reading them, which can leave the system half-upgraded or unbootable. Unlike
Debian/Ubuntu's `unattended-upgrades`, which applies small vetted security patches, Arch's
rolling updates aren't safe to apply blind. So: update manually and regularly, reading the news
first. This branch only automates the reminder.

Run it from your regular user.

## 1.0 Install pacman-contrib
```shell
sudo pacman -S pacman-contrib
```
Provides `checkupdates`, which lists pending updates using a temporary copy of the package
database. Unlike `pacman -Sy`, it leaves your real database untouched, so it's safe to run on a
schedule.

## 2.0 Create a Systemd Timer That Checks on a Schedule
#### Create the service that runs the check:
```shell
sudo nano /etc/systemd/system/checkupdates.service
```
```ini
[Unit]
Description=Check for pending pacman updates

[Service]
Type=oneshot
ExecStart=-/usr/bin/checkupdates
StandardOutput=journal
```
Runs `checkupdates` and logs the list of pending updates to the journal; it never installs
anything. The leading `-` stops systemd from counting `checkupdates`' "no updates" exit code as a
failure.

#### Create the timer:
```shell
sudo nano /etc/systemd/system/checkupdates.timer
```
```ini
[Unit]
Description=Run checkupdates daily

[Timer]
OnCalendar=daily
Persistent=true

[Install]
WantedBy=timers.target
```
Runs the check daily. `Persistent=true` runs a missed check at next boot.

#### Enable it:
```shell
sudo systemctl enable --now checkupdates.timer
```

#### See what it found:
```shell
journalctl -u checkupdates.service
```
Lists the pending updates from the last run - your cue to read the news and upgrade.

**Want a desktop notification instead?** Replace `ExecStart` with a script that pipes
`checkupdates` into `notify-send`, run as your own user, since notifications are per-session.
The details depend on your desktop environment.

## 3.0 Read the News Before You Actually Upgrade
```shell
sudo pacman -Syu
```
Before running this, check [archlinux.org/news](https://archlinux.org/news/) for anything posted
since your last upgrade. Two ways:

- **Manually:** read the page or its RSS feed before each upgrade.
- **`informant`** (AUR): a pacman hook that blocks `pacman -Syu` while there's an unread news
  entry, until you mark it read. Build and install it from the AUR:
  ```shell
  sudo pacman -S --needed binutils make gcc pkg-config fakeroot debugedit git
  git clone https://aur.archlinux.org/informant.git
  cd informant && makepkg -si
  ```
  The first line installs the AUR build tools (`--needed` skips any already installed);
  `makepkg -si` builds the package and installs it with its dependencies.

## Continue in the main guide
Continue at [29.0 System Configuration](../arch-linux-install-guide.md#290-system-configuration).
