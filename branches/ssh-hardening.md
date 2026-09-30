# Branch: SSH Hardening

The [main guide](../arch-linux-install-guide.md)'s 27.0 SSH Hardening: key-only login, no root
login, and `fail2ban`. It applies only if `systemctl is-enabled sshd 2>/dev/null` prints
`enabled`.

Run it from your regular user. Commands are shown with `sudo`; if `which sudo doas 2>/dev/null`
shows only `doas`, use `doas` instead.

**Before you start:** confirm you can already log in over SSH with your password from another
machine, so you have a fallback if key login fails.

## 1.0 Set Up Key-Based Login

#### On the machine you'll connect *from* (not the Arch machine), create a key pair if you don't have one:
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

## 2.0 Disable Password Authentication and Root Login
```shell
sudo nano /etc/ssh/sshd_config
```
Set these, uncommenting them if needed:
```conf
PasswordAuthentication no
PermitRootLogin no
```
Only key logins are accepted now, so a stolen or guessed password can't get in, and nobody can
log in directly as `root`.

#### Apply the change:
```shell
sudo systemctl restart sshd
```

#### Verify from another terminal (without closing your current session) that you can still log in:
```shell
ssh <your-username>@<your-hostname-or-ip>
```
Keep the current session open until this works, so you can still fix the config.

## 3.0 Install fail2ban (Brute-Force Protection)
```shell
sudo pacman -S fail2ban
```
`fail2ban` temporarily blocks IP addresses after repeated failed logins, which cuts down on
automated brute-force scans.

#### Enable the SSH jail:
```shell
printf '[sshd]\nenabled = true\nbackend = systemd\n' | sudo tee /etc/fail2ban/jail.local
```
Works in bash, zsh, and fish. `jail.local` overrides the packaged `jail.conf`, which package
updates may replace, and turns on its disabled SSH jail. `backend = systemd` reads login
attempts from the systemd journal, since a base Arch install has no `/var/log/auth.log`.

#### Enable and start it:
```shell
sudo systemctl enable --now fail2ban
```

#### Verify:
```shell
sudo fail2ban-client status sshd
```
Should show the `sshd` jail with a (probably empty) list of banned IPs.

## Continue in the main guide
Continue at [28.0 Update Hygiene](../arch-linux-install-guide.md#280-update-hygiene).
