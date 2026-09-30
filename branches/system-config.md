# Branch: System Quality-of-Life (Networking, Power, Time)

The [main guide](../arch-linux-install-guide.md)'s 29.0 System Configuration: three independent
alternatives to the main guide's defaults. Take any, none, or all.

Run it from your regular user.

## Network Management: NetworkManager

The main guide uses `iwd` (Wi-Fi) and `dhcpcd` (DHCP), managed from the command line.
NetworkManager handles wired and Wi-Fi in one service, remembers networks, reconnects
automatically, and is what desktop environments' network widgets (GNOME, KDE, XFCE) expect.
Switch if you want that widget to work, or prefer one tool for everything.

```shell
sudo pacman -S networkmanager
sudo systemctl disable --now iwd dhcpcd
sudo systemctl enable --now NetworkManager
```
Stops and disables `iwd`/`dhcpcd` first, since two network managers conflict. This drops your
connection until NetworkManager reconnects, so run it at the machine, not over SSH.

#### Connect to Wi-Fi with NetworkManager's CLI:
```shell
nmcli device wifi list
nmcli device wifi connect <your-ssid> password <your-wifi-password>
```
Desktop network widgets talk to the same service and show the same connections.

## Power Management (Mainly for Laptops)

A base Arch install does nothing special for battery life. Pick one of these, not both - they
conflict over the same settings.

**`power-profiles-daemon`:** Performance/Balanced/Power Saver profiles you switch from a desktop
environment's battery widget (GNOME's talks to it directly). Simple, with few settings.
```shell
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

**TLP:** tunes CPU scaling, USB autosuspend, disk power, and more automatically based on
AC/battery state, with many settings and no manual switching.
```shell
sudo pacman -S tlp
sudo systemctl enable --now tlp
```

## Time Sync: chrony (Alternative to systemd-timesyncd)

The main guide syncs time with `systemd-timesyncd`, which is enough for a typical machine.
`chrony` adds finer control: custom NTP servers, faster resync after suspend, and serving time
to other machines.

```shell
sudo pacman -S chrony
sudo systemctl disable --now systemd-timesyncd
sudo systemctl enable --now chronyd
```
Only one time-sync daemon should run, so this replaces `systemd-timesyncd` with `chronyd`.

#### Verify:
```shell
chronyc tracking
```

## Continue in the main guide
Continue at [30.0 Shell Configuration](../arch-linux-install-guide.md#300-shell-configuration).
