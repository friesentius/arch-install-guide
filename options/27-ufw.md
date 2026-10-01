# 27.0 Firewall: ufw

```shell
sudo pacman -S ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw enable
sudo systemctl enable --now ufw
sudo ufw status verbose
```
Blocks every unsolicited inbound connection while leaving outbound traffic alone, now and on
every boot. The status should show `Status: active`, `deny (incoming)`, and `allow (outgoing)`.

To make a service reachable later, allow its port:
```shell
sudo ufw allow <port>/tcp  # e.g. sudo ufw allow 8080/tcp
```

Continue at [28.0 SSH Server](../arch-linux-install-guide.md#280-ssh-server).
