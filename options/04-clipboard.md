# 4.0 List Disks: Copy/Paste on the Live ISO

```shell
systemctl start gpm
```
The live ISO already has `gpm`, the mouse server, installed but not running; this starts it. A
left-button drag selects text in any virtual console (switch with e.g. `Ctrl+Alt+F2`), and a
middle-click (right-click on a two-button mouse) pastes it, including into a different console.
Use it to copy a disk name or UUID shown by one command and paste it into the next, instead of
retyping it.

Continue at [5.0 Partition the Disk](../arch-linux-install-guide.md#50-partition-the-disk).
