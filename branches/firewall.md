# Branch: Firewall (ufw)

The [main guide](../arch-linux-install-guide.md)'s 26.0 Firewall: `ufw` blocks every unsolicited
inbound connection while leaving outbound traffic alone. `ufw` is a simple frontend that writes
the kernel's packet-filter rules for you; writing `nftables` rules directly gives more control
but is beyond this guide.

Run it from your regular user. Commands are shown with `sudo`; if `which sudo doas 2>/dev/null`
shows only `doas`, use `doas` instead.

## 1.0 Install ufw
```shell
sudo pacman -S ufw
```

## 2.0 Set Default Policies
```shell
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
Any inbound service you want reachable now needs an explicit `allow` rule.

## 3.0 Allow SSH Through
Skip this unless `systemctl is-enabled sshd 2>/dev/null` prints `enabled`.
```shell
sudo ufw allow ssh
```
Allows port 22/tcp. If you moved SSH to another port, use `sudo ufw allow <your-port>/tcp`
instead.

## 4.0 Enable ufw
```shell
sudo ufw enable
sudo systemctl enable --now ufw
```
Turns the firewall on now, and loads its rules on every boot.

## 5.0 Verify
```shell
sudo ufw status verbose
```
Should show `Status: active`, `deny (incoming)`, `allow (outgoing)`, and your SSH rule if you
added one.

## Allowing More Services Later
```shell
sudo ufw allow <port-or-service>/tcp  # e.g. sudo ufw allow 8080/tcp
```
Add one rule per service you want reachable from the network; everything else stays blocked.

## Continue in the main guide
Continue at [27.0 SSH Hardening](../arch-linux-install-guide.md#270-ssh-hardening).
