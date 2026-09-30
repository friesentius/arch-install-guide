# Arch Linux Install Guide

<!-- Created by https://gitlab.com/runit25/infosphere -->

A from-scratch Arch Linux install guide, written as a sequence of copy-pasteable commands with
plain-language explanations of what each step does and why. It targets a UEFI system with a
single disk.

## Getting started

Start with the [main install guide](arch-linux-install-guide.md). It's laid out as **one
numbered path** from a live ISO to a fully configured system - keyboard layout, disk layout, base
install, system configuration, bootloader, first boot, then post-install polish. Every step shows
this guide's default choice inline.

## Choosing your own route

At every step where Arch offers a real alternative - a different disk layout, encryption, a
different bootloader, shell, or privilege-escalation tool, or a post-install add-on like a
firewall - the main guide's step calls it out with a **"Want X instead?"** link into
[`branches/`](branches/README.md). Take the branch, and it ends with a link back to the exact
step of the main guide you left. You can take as many of these scenic routes as you like and
still end up at the same destination: a working, bootable Arch system configured the way you
want it. See [`branches/README.md`](branches/README.md) for the full branch index and how each
one forks from and rejoins the main path.

If you're not sure whether you need a branch, you probably don't - taking the default at every
step gets you a complete, working install.

## Conventions used throughout

- A value wrapped in angle brackets, like `/dev/<your-disk>` or `<your-username>`, is a
  placeholder: replace it with the real value for your system.
- An unbracketed example value (like `wlan0` or a hostname) is just an example or a name the
  guide invents along the way - check the relevant command's output for what's actually true on
  your system before continuing.
- At genuine choice points (keyboard layout, locale, text editor, and so on) the guide picks a
  sensible default (US) and calls out common alternatives inline - including non-US layouts like
  Dvorak and Colemak - rather than silently picking one for you. Keyboard layout is set as the
  very first step in the guide, before anything else is typed, and carried through to the
  installed system's console and, if you take the disk-encryption branch, its LUKS passphrase
  prompt.
- Larger choices - a different disk layout, bootloader, shell, privilege-escalation tool, or a
  post-install add-on - are branches: separate files in [`branches/`](branches/README.md), each
  linked from the main guide step where the choice is made, each ending with a link back to
  where you rejoin.
