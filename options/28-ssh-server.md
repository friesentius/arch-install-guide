# 28.0 SSH Server: OpenSSH

Installs OpenSSH so you can log in from other machines, then locks it down with key-only login,
no root login, and `fail2ban`.

#### Install and start OpenSSH:
```shell
sudo pacman -S openssh
sudo systemctl enable --now sshd
command -v ufw >/dev/null && sudo ufw allow ssh
```
Starts the SSH server now and on every boot. The last line opens port 22 in the `ufw` firewall,
and does nothing if `ufw` isn't installed.

Find the machine's address with `ip -brief address`, then confirm you can log in with your
password from another machine, so you have a fallback if key login fails:
```shell
ssh <your-username>@<your-hostname-or-ip>  # e.g. archie@192.168.1.50
```

#### On the machine you'll connect *from*, create a key pair if you don't have one:
```shell
ssh-keygen -t ed25519
```
Accept the default file location. A passphrase is optional and protects the key itself.

#### Copy your public key to the Arch machine:
```shell
ssh-copy-id <your-username>@<your-hostname-or-ip>  # e.g. archie@192.168.1.50
```
Adds your key to `~/.ssh/authorized_keys` on the Arch machine. You'll enter your password one
last time.

#### Test key-based login before continuing:
```shell
ssh <your-username>@<your-hostname-or-ip>
```
It should log you in without a password prompt. Don't continue until it does.

#### Disable password authentication and root login:
```shell
sudo nano /etc/ssh/sshd_config
```
Set these, uncommenting them if needed:
```conf
PasswordAuthentication no
PermitRootLogin no
```
Then apply the change:
```shell
sudo systemctl restart sshd
```
Only key logins are accepted now, so a stolen or guessed password can't get in, and nobody can
log in directly as `root`. Without closing your current session, confirm from another terminal
that you can still log in:
```shell
ssh <your-username>@<your-hostname-or-ip>
```

#### Install fail2ban (brute-force protection):
```shell
sudo pacman -S fail2ban
printf '[sshd]\nenabled = true\nbackend = systemd\n' | sudo tee /etc/fail2ban/jail.local
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```
`fail2ban` temporarily blocks IP addresses after repeated failed logins. `jail.local` turns on
its SSH jail without editing the packaged `jail.conf`, which package updates may replace, and
`backend = systemd` reads login attempts from the systemd journal, since a base Arch install has
no `/var/log/auth.log`. The status should show the `sshd` jail with a (probably empty) list of
banned IPs.

Continue at [29.0 Update Hygiene](../arch-linux-install-guide.md#290-update-hygiene).
