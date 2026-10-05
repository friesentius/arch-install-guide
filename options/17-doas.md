# 17.0 Configure Privilege Escalation: doas

`opendoas` is a port of OpenBSD's `doas`: a much smaller codebase and simpler config than `sudo`,
but less common and less configurable.
```shell
pacman -S opendoas
```

#### Create /etc/doas.conf:
```shell
nano /etc/doas.conf
```
Write the following into the file:
```conf
# /etc/doas.conf
permit persist <your-username>  # e.g. archie
```
```shell
chmod 600 /etc/doas.conf
ln -s /usr/bin/doas /usr/local/bin/sudo
```
Lets your user run any command as root with `doas <command>`. `persist` skips the password prompt
for five minutes after you enter it, and `chmod 600` makes the file readable only by root. The
link makes `sudo <command>` run `doas <command>`, so the `sudo` commands in the rest of this
guide, your muscle memory, and simple scripts all work unchanged; `sudo`-only options, such as
`sudo -E`, won't.

Continue at [18.0 Exit Chroot](../arch-linux-install-guide.md#180-exit-chroot).
