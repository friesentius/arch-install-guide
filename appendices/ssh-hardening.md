# Appendix: SSH Hardening

**This appendix only applies if you installed `openssh` and enabled `sshd`** back in the core
guide's Base Installation / 7.0 Enable Networking Services steps. If you didn't install
openssh, there's no SSH service to harden and nothing here applies to you.

This is one of the "Next Steps" from the end of the
[core install guide](../arch-linux-install-guide.md): it doesn't replace anything there, it's an
optional hardening pass for after you have a bootable, logged-in system with SSH access working.
Run these commands from your regular user account (with `sudo`).

**Before you start:** confirm you can already log in over SSH using your password, from another
machine, before disabling password authentication below - if key-based login doesn't work for
some reason, you want a working fallback, not a locked door.

## 1.0 Set Up Key-Based Login

#### On the machine you'll be connecting *from* (not the Arch box), generate a key pair if you don't already have one:
```shell
ssh-keygen -t ed25519
```
`ed25519` is a modern, fast, secure key type - a good default over the older `rsa`. Accept the
default file location; set a passphrase if you want the key itself password-protected.

#### Copy your public key to the Arch machine:
```shell
ssh-copy-id <your-username>@<your-hostname-or-ip>  # e.g. archie@192.168.1.50
```
Appends your public key to `~/.ssh/authorized_keys` on the Arch machine, over SSH (you'll be
prompted for your password one last time). Replace `<your-username>` with the username you
created in the core guide's Add User step, and `<your-hostname-or-ip>` with however you reach
the machine on your network.

#### Test key-based login before continuing:
```shell
ssh <your-username>@<your-hostname-or-ip>
```
Should log you in without prompting for a password. Don't continue to the next step until this
works.

## 2.0 Disable Password Authentication and Root Login
```shell
sudo nano /etc/ssh/sshd_config
```
Find and set (uncommenting if necessary):
```conf
PasswordAuthentication no
PermitRootLogin no
```
`PasswordAuthentication no` means only key-based logins are accepted from here on - a stolen or
guessed password can no longer get anyone in over SSH. `PermitRootLogin no` disables logging in
as `root` over SSH entirely; log in as your regular user and use `sudo` instead, which is both
more secure and gives you an audit trail of who ran what.

#### Apply the change:
```shell
sudo systemctl restart sshd
```

#### Verify from another terminal (without closing your current session) that you can still log in:
```shell
ssh <your-username>@<your-hostname-or-ip>
```
Keep your current SSH session open until this succeeds - if something's wrong with the new
config, you still have a way in to fix it.

## 3.0 Install fail2ban (Brute-Force Protection)
```shell
sudo pacman -S fail2ban
```
`fail2ban` watches for repeated failed login attempts and temporarily firewalls off the
offending IP address - it doesn't stop a targeted attack, but it cuts down the constant
background noise of automated SSH brute-force scans hitting the internet at large.

#### Create a local jail override enabling the SSH jail:
```shell
sudo tee /etc/fail2ban/jail.local <<'EOF'
[sshd]
enabled = true
backend = systemd
EOF
```
`fail2ban`'s packaged `jail.conf` defines an SSH jail but leaves it disabled by default;
`jail.local` overrides just the settings you specify, without you having to edit (and risk
losing on the next package update) the packaged config file directly. `backend = systemd` tells
`fail2ban` to read login attempts straight from the systemd journal, since a base Arch install
has no traditional `/var/log/auth.log`-style syslog file for it to watch instead.

#### Enable and start it:
```shell
sudo systemctl enable --now fail2ban
```

#### Verify:
```shell
sudo fail2ban-client status sshd
```
Should show the `sshd` jail active, with a (probably empty, for now) list of currently banned
IPs.

## Continue
This appendix has no further steps of its own. Return to the core guide's
[Next Steps](../arch-linux-install-guide.md#next-steps) for the other optional appendices.
