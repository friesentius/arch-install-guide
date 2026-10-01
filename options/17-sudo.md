# 17.0 Configure Privilege Escalation: sudo (default)

```shell
pacman -S sudo
EDITOR=nano visudo
```
`visudo` edits `/etc/sudoers` and checks its syntax before saving, since a broken file can lock
you out of root. Uncomment this line, then save and exit:
```conf
%wheel ALL=(ALL:ALL) ALL
```
Members of `wheel`, including your user, can now run any command as root with `sudo`.

Continue at [18.0 Exit Chroot](../arch-linux-install-guide.md#180-exit-chroot).
