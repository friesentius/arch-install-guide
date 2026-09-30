# Branch: Alternate Privilege Escalation (opendoas)

Replaces the [main guide](../arch-linux-install-guide.md)'s 20.0 Configure Privilege Escalation,
using `opendoas` instead of `sudo`. `opendoas` is a port of OpenBSD's `doas`: a much smaller
codebase and a simpler config than `sudo`, but less common and less configurable. Run it inside
the chroot.

## 20.0 Configure Privilege Escalation (doas)
```shell
pacman -S opendoas
echo "permit persist <your-username>" > /etc/doas.conf  # e.g. archie
chmod 600 /etc/doas.conf
```
Lets your user run any command as root with `doas <command>`. `persist` skips the password prompt
for five minutes after you enter it. `chmod 600` makes the file readable only by root.

#### Make `sudo` run `doas`:
```shell
ln -s /usr/bin/doas /usr/local/bin/sudo
```
`sudo` itself isn't installed on this route. This link makes `sudo <command>` run
`doas <command>`, so the `sudo` commands in the rest of this guide, your muscle memory, and
simple scripts all work unchanged. `sudo`-only options, such as `sudo -E`, won't.

## Continue in the main guide
Continue at [21.0 Exit Chroot](../arch-linux-install-guide.md#210-exit-chroot).
