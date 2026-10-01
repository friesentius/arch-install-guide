# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install guide for a UEFI system with a single disk: copy-pasteable
commands, each with a plain-language explanation.

## Getting started

Follow the [install guide](arch-linux-install-guide.md) from top to bottom: one numbered sequence
of steps from the live ISO to a configured system - keyboard layout, disk setup, base install,
system configuration, bootloader, first boot, then optional post-install steps.

## Choosing your own install

The main guide holds only the steps. The options live in [`options/`](options/README.md), one
doc each, in two shapes:

- **Choice steps** (disk setup, bootloader, privilege escalation) list their options, one marked
  **(default)**. Pick one and follow its doc.
- **Optional steps** (Wi-Fi, and every post-install add-on) link to a short doc you can take or
  skip.

Every doc ends by sending you to the next step, so you never have to remember which options you
took. The disk-setup doc you pick saves its boot settings for the later steps, so those steps are
the same for everyone. Taking every default gives a complete install.

## Conventions

- A value in angle brackets, like `/dev/<your-disk>` or `<your-username>`, is a placeholder for
  your own value.
- Unbracketed values (like `wlan0`) are examples or names the guide creates; check the relevant
  command's output for yours.
- Defaults are US (keyboard, locale), with common alternatives such as Dvorak and Colemak listed
  where the choice is made.
