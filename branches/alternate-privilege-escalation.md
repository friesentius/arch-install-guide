# Branch: Alternate Privilege Escalation (opendoas)

Replaces the [main guide](../arch-linux-install-guide.md)'s 19.0 Configure Privilege Escalation,
using `opendoas` instead of `sudo`. `opendoas` is a port of OpenBSD's `doas`: a much smaller
codebase and a simpler config than `sudo`, but less common and less configurable. Run it inside
the chroot.

## 1.0 Install opendoas
```shell
pacman -S opendoas
```

## 2.0 Allow Your User to Run Commands as Root
```shell
echo "permit persist <your-username>" > /etc/doas.conf  # e.g. archie
chmod 600 /etc/doas.conf
```
Lets your user run any command as root with `doas <command>`. `persist` skips the password prompt
for five minutes after you enter it. `chmod 600` makes the file readable only by root.

## 3.0 Optional: Make `sudo` Run `doas`
```shell
ln -s /usr/bin/doas /usr/local/bin/sudo
```
`sudo` itself isn't installed on this route. This link lets muscle memory and simple scripts
that call `sudo <command>` keep working, whichever shell you use. `sudo`-only options won't.

## Continue in the main guide
Continue at [20.0 Install and Configure systemd-boot](../arch-linux-install-guide.md#200-install-and-configure-systemd-boot).
