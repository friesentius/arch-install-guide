# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install guide, written as a sequence of copy-pasteable commands with
plain-language explanations of what each step does and why. It targets a UEFI system with a
single disk.

## Getting started

Start with the [core install guide](arch-linux-install-guide.md). It walks through a full,
minimal install: partitioning, base packages, locale/network configuration, and the
systemd-boot bootloader, using the simplest common setup (one EFI partition, one ext4 root
partition, a swapfile).

## Appendices

Once you have the core guide's basic install working, or if you want a different disk layout or
bootloader from the start, see [`appendices/README.md`](appendices/README.md) for optional and
specialized topics (disk layout and encryption, bootloader choice, security hardening, system
configuration) and how each one relates to the core guide.

## Conventions used throughout

- A value wrapped in angle brackets, like `/dev/<your-disk>` or `<your-username>`, is a
  placeholder: replace it with the real value for your system.
- An unbracketed example value (like `wlan0` or a hostname) is just an example or a name the
  guide invents along the way - check the relevant command's output for what's actually true on
  your system before continuing.
- At genuine choice points (keyboard layout, locale, text editor, and so on) the guide picks a
  sensible default and calls out common alternatives inline, rather than silently picking one
  for you.
