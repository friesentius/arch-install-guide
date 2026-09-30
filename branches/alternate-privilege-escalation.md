# Branch: Alternate Privilege Escalation (opendoas)

This branch is a swap-in replacement for the [main guide](../arch-linux-install-guide.md)'s
19.0 Configure Privilege Escalation step. Use it if you'd rather use `opendoas` instead of
`sudo`; everything else in the main guide continues unchanged.

**Use this branch if:** you want a smaller, simpler privilege-escalation tool than `sudo` -
`opendoas` (a maintained fork of OpenBSD's `doas`) has a much smaller codebase and a terser
config file, at the cost of being less common and less configurable than `sudo`.

## 1.0 Install opendoas
```shell
pacman -S opendoas
```
Installs `opendoas` and its `doas` command.

## 2.0 Allow Your User to Run Commands as Root
```shell
echo "permit persist <your-username>" > /etc/doas.conf  # e.g. archie
chmod 600 /etc/doas.conf
```
`doas.conf` is opendoas's permission list; this grants your user (replace `<your-username>`
with the username you created in the main guide's 18.0 Add User step) permission to run commands as
root via `doas <command>`. `persist` caches successful authentication for five minutes, so
you're not re-prompted for a password on every single `doas` call. `chmod 600` keeps the config
file readable only by root, since a world-readable `doas.conf` would leak who has root access.

## 3.0 Optional: Add a sudo Alias
```shell
echo "alias sudo=doas" >> /home/<your-username>/.bashrc  # e.g. /home/archie/.bashrc
```
Because this branch replaces the main guide's `sudo` step entirely, the real `sudo` command
isn't installed. This is purely a convenience if your muscle memory (or scripts) expect `sudo`:
it lets you type the more familiar `sudo <command>` and have it actually run `doas <command>`.
Replace `<your-username>` with your username.

## Continue in the main guide
Privilege escalation is set up. Continue with the main guide's
[20.0 Install and Configure systemd-boot](../arch-linux-install-guide.md#200-install-and-configure-systemd-boot).
