# Appendix: System Quality-of-Life (Networking, Power, Time)

This is one of the "Next Steps" from the end of the
[core install guide](../arch-linux-install-guide.md): it doesn't replace anything there, it's a
set of independent, optional alternatives to defaults the core guide already set up. Pick
whichever of these (if any) fit your machine and usage - none are required, and they don't
depend on each other.

## Network Management: NetworkManager

The core guide sets up networking with `iwd` (Wi-Fi) and `dhcpcd` (DHCP), controlled directly
via `systemctl` and `iwctl` - simple and lightweight, but purely command-line, with no built-in
concept of "known networks" beyond what you type. **NetworkManager** is a more desktop-friendly
alternative: one service that manages both wired and Wi-Fi, remembers networks, auto-reconnects,
and is what most desktop environments' network widgets (GNOME, KDE, XFCE, and so on) expect to
talk to.

**Switch to NetworkManager if:** you're setting up a desktop environment and want its network
applet/widget to work out of the box, or you just prefer one tool that manages everything
instead of `iwctl` plus `dhcpcd`.

```shell
sudo pacman -S networkmanager
sudo systemctl disable iwd dhcpcd
sudo systemctl enable --now NetworkManager
```
Installs NetworkManager, disables the core guide's `iwd`/`dhcpcd` services (running two network
managers against the same interfaces at once causes conflicts), and enables/starts
NetworkManager instead.

#### Connect to Wi-Fi with NetworkManager's CLI:
```shell
nmcli device wifi list
nmcli device wifi connect <your-ssid> password <your-wifi-password>
```
`nmcli` is NetworkManager's command-line tool - a desktop environment's GUI network widget talks
to the same underlying service and will show the same networks/connections. Replace
`<your-ssid>` and `<your-wifi-password>` with your actual network name and password.

## Power Management (Mainly for Laptops)

Neither the core guide nor a minimal Arch install does anything special for battery life out of
the box. Two common options, mainly relevant if this is a laptop:

**`power-profiles-daemon`:** a simple daemon exposing "Performance"/"Balanced"/"Power Saver"
profiles, switchable from a desktop environment's battery widget (GNOME's, for instance, talks
to it directly). Simpler and less configurable than TLP below.
```shell
sudo pacman -S power-profiles-daemon
sudo systemctl enable --now power-profiles-daemon
```

**TLP:** a more thorough, configurable power-management tool with many tunables (CPU frequency
scaling, USB autosuspend, disk power management, and more) applied automatically based on
AC/battery state, without you needing to switch profiles manually. More capable, but don't run
it alongside `power-profiles-daemon` - the two conflict over the same settings.
```shell
sudo pacman -S tlp
sudo systemctl enable --now tlp
```

**Pick one, not both:** if you want simple profile-switching integrated with your desktop
environment, use `power-profiles-daemon`; if you want more thorough, mostly-automatic tuning,
use TLP.

## Time Sync: chrony (Alternative to systemd-timesyncd)

The core guide already enables NTP time sync via `timedatectl set-ntp true`, which uses
`systemd-timesyncd` - a simple SNTP client, fine for keeping a typical desktop/laptop's clock
accurate. **`chrony`** is a more capable alternative if you want finer control over time sync
(multiple/custom NTP servers, faster resync after suspend, acting as a local NTP server for
other machines) - most single-machine setups don't need it.

```shell
sudo pacman -S chrony
sudo systemctl disable systemd-timesyncd
sudo systemctl enable --now chronyd
```
Installs chrony, disables the core guide's `systemd-timesyncd` (only one time-sync daemon
should run at a time), and enables/starts `chronyd` instead.

#### Verify:
```shell
chronyc tracking
```

## Continue
This appendix has no further steps of its own. Return to the core guide's
[Next Steps](../arch-linux-install-guide.md#next-steps) for the other optional appendices.
