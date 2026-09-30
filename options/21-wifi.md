# 21.0 Log In: Connect to Wi-Fi

```shell
iwctl device list                        # Identify interface (e.g., wlan0)
iwctl station wlan0 connect <your-ssid>  # e.g. connect MyHomeWiFi
```
Replace `wlan0` with the interface `device list` shows, and `<your-ssid>` with your network name.
`iwd` remembers the network from now on.

Continue at [22.0 Verify Installation](../arch-linux-install-guide.md#220-verify-installation).
