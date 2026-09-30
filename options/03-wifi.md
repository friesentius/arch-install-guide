# 3.0 Connect to the Internet: Wi-Fi

```shell
iwctl
device list                       # Identify interface (e.g., wlan0)
station wlan0 scan                # Scan networks
station wlan0 get-networks        # List networks
station wlan0 connect <your-ssid> # e.g. connect MyHomeWiFi
exit
```
`iwctl` is the live ISO's Wi-Fi client. Replace `wlan0` with the interface `device list` shows
(e.g. `wlp3s0`), and `<your-ssid>` with your network name. Then check the connection:
```shell
ping -c 3 archlinux.org
```

Continue at [4.0 List Disks](../arch-linux-install-guide.md#40-list-disks).
