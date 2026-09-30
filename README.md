# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install guide for a UEFI system with a single disk: copy-pasteable
commands, each with a plain-language explanation.

## Getting started

Follow the [install guide](arch-linux-install-guide.md) from top to bottom: one numbered sequence
of steps from the live ISO to a configured system - keyboard layout, disk setup, base install,
system configuration, bootloader, first boot, then optional post-install steps.

## Choosing your own install

Each step is either a single set of commands or a short list of options, one marked
**(default)**. An option holds only the commands that differ for that step, so you pick one, run
it, and carry on with the next step. Where an earlier choice (such as encrypting the disk)
changes a later step, that step's options are named by what's true of your system, like
"Encrypted disk (LUKS)", so you never have to remember which options you took. Taking every
default gives a complete install.

The couple of options too long to sit inline live in [`branches/`](branches/README.md), and each
returns you to the next step.

## Conventions

- A value in angle brackets, like `/dev/<your-disk>` or `<your-username>`, is a placeholder for
  your own value.
- Unbracketed values (like `wlan0`) are examples or names the guide creates; check the relevant
  command's output for yours.
- Defaults are US (keyboard, locale), with common alternatives such as Dvorak and Colemak listed
  where the choice is made.
