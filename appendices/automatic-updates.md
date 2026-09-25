# Appendix: Update Hygiene (Scheduled Checks, Not Unattended Upgrades)

This is one of the "Next Steps" from the end of the
[core install guide](../arch-linux-install-guide.md): it doesn't replace anything there. It's
about how to stay on top of updates on a rolling-release distro without either (a) never
updating, or (b) blindly automating `pacman -Syu` the way you might set up
`unattended-upgrades` on Ubuntu/Debian.

**Why not just automate `pacman -Syu` on a timer?** On Arch, that's a real risk, not caution for
its own sake:
- Some upgrades require **manual intervention** - a config file format changes, a package is
  split or renamed, a service needs a manual migration step - and skipping that can leave the
  system half-upgraded or unbootable.
- Arch announces breaking changes on [archlinux.org/news](https://archlinux.org/news/) *before*
  they land in a routine upgrade, specifically so you can read the relevant entry and do
  whatever manual step it describes first. An unattended upgrade skips that entirely.
- Arch is a rolling release with a much faster, less curated update cadence than a point-release
  distro's tested-as-a-batch security updates - the assumption behind tools like Debian/Ubuntu's
  `unattended-upgrades` (small, individually-vetted patches) doesn't hold here.

The actual recommended practice on Arch is: **update manually, regularly, and read the news
first.** This appendix only makes "regularly" easier to keep up with, via a reminder - not by
removing the "read the news first" part.

## 1.0 Install pacman-contrib
```shell
sudo pacman -S pacman-contrib
```
Provides `checkupdates`, which checks for available updates against a temporary copy of the
package database, without touching your system's actual sync database the way `pacman -Sy` on
its own would (which is unsafe to run without also upgrading in the same step). That makes it
safe to run on a schedule purely to check.

## 2.0 Create a Systemd Timer That Checks (and Notifies) on a Schedule
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
This defines a one-shot job that runs `checkupdates` and sends its output (the list of
upgradable packages, if any) to the systemd journal - it deliberately does not run
`pacman -Syu` or apply anything. The leading `-` in `ExecStart` tells systemd not to treat
`checkupdates`' "nothing to update" exit code as a service failure.

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
`OnCalendar=daily` runs the check once a day; `Persistent=true` catches up on a missed run (e.g.
the machine was off at the scheduled time) the next time it boots.

#### Enable it:
```shell
sudo systemctl enable --now checkupdates.timer
```

#### See what it found:
```shell
journalctl -u checkupdates.service
```
Lists the package names and version bumps found on the last run, if any - your cue to go read
the news (next section) and then upgrade manually when you have a moment.

**Want a desktop notification instead of a journal entry?** If you're running a desktop
environment, swap `ExecStart` above for a wrapper script that pipes `checkupdates`' output into
`notify-send` (run as your own user, not root, since desktop notifications are per-session) -
the exact setup depends on your desktop environment and is out of scope here.

## 3.0 Read the News Before You Actually Upgrade
```shell
sudo pacman -Syu
```
Before running this (whenever the timer above tells you there's something to install), check
[archlinux.org/news](https://archlinux.org/news/) for anything posted since your last upgrade -
that's where manual-intervention steps get announced. Two ways to do this:

- **Manually:** check the page (or its RSS feed) in a browser before every upgrade.
- **`informant`** (AUR): a pacman hook that reads the same news feed and **blocks**
  `pacman -Syu` with an error if there's an unread entry relevant to your system, until you
  acknowledge having read it. This turns "remember to check the news" into something pacman
  enforces for you:
  ```shell
  # from the AUR, e.g. using an AUR helper such as paru or yay
  paru -S informant
  ```
  (Installing an AUR helper itself is covered in the
  [graphics-and-extras appendix](graphics-and-extras.md)'s AUR build-dependencies section.)

## Continue
This appendix has no further steps of its own. Return to the core guide's
[Next Steps](../arch-linux-install-guide.md#next-steps) for the other optional appendices.
