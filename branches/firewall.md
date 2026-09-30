# Branch: Firewall (ufw)

This branch is the [main guide](../arch-linux-install-guide.md)'s 26.0 Firewall step: it's an
optional step for after you have a bootable, logged-in system. It sets up a firewall with a
default-deny-inbound posture: nothing gets in unless you explicitly allow it, everything can
still get out.

**Use this branch if:** you want a basic, sane firewall without hand-writing packet-filter
rules yourself. This guide uses `ufw` ("Uncomplicated Firewall") - a simple command-line
frontend that's the easiest way to get default-deny-inbound working correctly. Run these
commands from your regular user account, after logging into your installed system. They're
shown with `sudo`; check `which sudo doas 2>/dev/null` first, and replace `sudo` with `doas` in
every command below if that's what's installed instead.

**A note on what's underneath:** `ufw` (and `firewalld`, another common frontend) both work by
generating rules for `nftables` (or, on older systems, `iptables`) - the kernel's actual
packet-filtering framework. Writing `nft`/`iptables` rules directly gives you more control, but
it's a much bigger topic than this guide covers; `ufw`'s simpler `allow`/`deny` model is enough
for a typical single-machine setup.

## 1.0 Install ufw
```shell
sudo pacman -S ufw
```

## 2.0 Set Default Policies
```shell
sudo ufw default deny incoming
sudo ufw default allow outgoing
```
This is the default-deny-inbound posture: by default, no unsolicited inbound connection is
accepted, while your own outbound connections (web browsing, package downloads, etc.) are
unaffected. Any inbound service you actually want reachable needs an explicit `allow` rule,
like the SSH one below.

## 3.0 Allow SSH Through
Check now whether this applies to you: `systemctl is-enabled sshd 2>/dev/null`. If it doesn't
print `enabled`, skip this step entirely - an inbound rule for a service that isn't running
doesn't help you, and it's one less thing to think about.
```shell
sudo ufw allow ssh
```
`ufw allow ssh` looks up the standard SSH port (22/tcp) from `/etc/services`; if you've moved
SSH to a nonstandard port, use `sudo ufw allow <your-port>/tcp` instead.

## 4.0 Enable ufw
```shell
sudo systemctl enable --now ufw
```
Enables the `ufw` service so your rules load on every boot, and starts it immediately.

## 5.0 Verify
```shell
sudo ufw status verbose
```
Should show status `active`, default policies of `deny (incoming)` / `allow (outgoing)`, and
(if you added it) a rule allowing SSH.

## Allowing More Services Later
```shell
sudo ufw allow <port-or-service>/tcp  # e.g. sudo ufw allow 8080/tcp
```
Add one `allow` rule per additional service you want reachable from the network (a web server,
a game server, etc.) - anything without an explicit rule stays blocked by the default-deny
policy.

## Continue in the main guide
This branch has no further steps of its own. Continue with the main guide's
[27.0 SSH Hardening](../arch-linux-install-guide.md#270-ssh-hardening).
