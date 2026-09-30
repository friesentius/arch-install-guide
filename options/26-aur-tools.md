# 26.0 AUR Build Tools

```shell
sudo pacman -S --needed binutils make gcc pkg-config fakeroot debugedit git
```
The AUR (Arch User Repository) ships build recipes (`PKGBUILD`s), not binaries, so installing
from it means compiling locally. This installs the build toolchain most `PKGBUILD`s expect, plus
`git` to fetch them.

Continue at [27.0 Firewall](../arch-linux-install-guide.md#270-firewall).
