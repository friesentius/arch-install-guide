# Appendix: Firewall (ufw)

This appendix is one of the "Next Steps" from the end of the
[core install guide](../arch-linux-install-guide.md): it doesn't replace anything in the core
guide, it's an optional step for after you have a bootable, logged-in system. It sets up a
firewall with a default-deny-inbound posture: nothing gets in unless you explicitly allow it,
everything can still get out.

**Use this appendix if:** you want a basic, sane firewall without hand-writing packet-filter
rules yourself. This guide uses `ufw` ("Uncomplicated Firewall") - a simple command-line
frontend that's the easiest way to get default-deny-inbound working correctly. Run these
commands from your regular user account (with `sudo`), after logging into your installed
system.

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

## 3.0 Allow SSH Through (if you installed openssh)
```shell
sudo ufw allow ssh
```
Only needed if you installed `openssh` and enabled `sshd` back in the core guide's Base
Installation / 7.0 Enable Networking Services steps. `ufw allow ssh` looks up the standard SSH
port (22/tcp) from `/etc/services`; if you've moved SSH to a nonstandard port, use
`sudo ufw allow <your-port>/tcp` instead. Skip this step entirely if you didn't install
openssh - an inbound rule for a service that isn't running doesn't help you, and it's one less
thing to think about.

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

## Continue
This appendix has no further steps of its own. Return to the core guide's
[Next Steps](../arch-linux-install-guide.md#next-steps) for the other optional appendices.
