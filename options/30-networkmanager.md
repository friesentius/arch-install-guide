# 30.0 Network Manager: Switch to NetworkManager

NetworkManager handles wired and Wi-Fi in one service, remembers networks, reconnects
automatically, and is what desktop environments' network widgets (GNOME, KDE, XFCE) expect.
```shell
sudo pacman -S networkmanager
sudo systemctl disable --now iwd dhcpcd
sudo systemctl enable --now NetworkManager
```
Stops and disables `iwd`/`dhcpcd` first, since two network managers conflict. This drops your
connection until NetworkManager reconnects, so run it at the machine, not over SSH. Wired
connections come back by themselves; for Wi-Fi:
```shell
nmcli device wifi list
nmcli device wifi connect <your-ssid> password <your-wifi-password>
```

Continue at [31.0 Power Management](../arch-linux-install-guide.md#310-power-management).
