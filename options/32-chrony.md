# 32.0 Time Sync: Switch to chrony

`chrony` gives finer control than `systemd-timesyncd`: custom NTP servers, faster resync after
suspend, and serving time to other machines.
```shell
sudo pacman -S chrony
sudo systemctl disable --now systemd-timesyncd
sudo systemctl enable --now chronyd
chronyc tracking
```
Only one time-sync daemon should run, so this replaces `systemd-timesyncd` with `chronyd`.

Continue at [33.0 Shell](../arch-linux-install-guide.md#330-shell).
