# 25.0 32-bit Packages: Enable multilib

```shell
sudo nano /etc/pacman.conf
```
Uncomment this section:
```conf
[multilib]
Include = /etc/pacman.d/mirrorlist
```
Then refresh the package databases and upgrade:
```shell
sudo pacman -Syu
```

Continue at [26.0 AUR Build Tools](../arch-linux-install-guide.md#260-aur-build-tools).
