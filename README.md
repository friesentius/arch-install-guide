# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install guide for a UEFI system with a single disk: copy-pasteable
commands, each with a plain-language explanation.

## Getting started

Follow the [main install guide](arch-linux-install-guide.md): one numbered path from the live ISO
to a configured system - keyboard layout, disk layout, base install, system configuration,
bootloader, first boot, then optional post-install steps. Each step shows a default inline.

## Choosing your own route

Where Arch offers a real alternative (LVM or encryption, Limine, doas, zsh/fish, or a
post-install add-on like a firewall), the step links to a branch in
[`branches/`](branches/README.md). A branch spells out every step its choice changes, in order,
then sends you back to the main guide. You never need to remember which branches you took: just
keep reading. Taking every default also gives you a complete install.

## Conventions

- A value in angle brackets, like `/dev/<your-disk>` or `<your-username>`, is a placeholder for
  your own value.
- Unbracketed values (like `wlan0`) are examples or names the guide creates; check the relevant
  command's output for yours.
- Defaults are US (keyboard, locale), with common alternatives such as Dvorak and Colemak listed
  where the choice is made.
