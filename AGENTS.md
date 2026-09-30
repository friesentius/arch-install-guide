# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Documentation-only repo (no code, CI, or tests). "Correctness" means the steps work on real
  Arch Linux and the Markdown links resolve. Check package/hook facts against a real Arch system
  (`pacman -Si`, `/usr/lib/initcpio/install/`) rather than memory.
- Layout: `arch-linux-install-guide.md` is the default route only, with globally numbered steps
  (`1.0`-`31.0`; post-install `25.0`+ each default to "skip"). `branches/` holds every
  alternative. `branches/README.md`'s table is the source of truth for fork/rejoin points; keep it
  and `README.md` in sync with the file set and step numbers.
- "Fog" rule (the captain's core requirement): a reader never needs to know or check which path
  they are on - each paragraph is actionable on its own.
  - A branch contains every step that differs for its choice, in order, including steps identical
    to the main guide (duplicate them, with the main guide's step numbers), and rejoins at the
    first main step after which nothing differs. The main guide has no per-branch variants and
    no "which one is yours" checks.
  - Rejoin lines are plain: `Continue at [N.0 Title](../arch-linux-install-guide.md#n0-title)`,
    no caveats or reminders.
  - Before duplicating, shrink a fork's reach: order steps so fork-dependent ones come first
    (swap/initramfs/bootloader sit before hostname/users in `9.0`-`15.0`), or neutralize the
    effect (doas branch symlinks `sudo` to `doas`; shell switching lives at `30.0`; SSH is a
    post-install step; `nano` is always installed; self-adapting commands like
    `command -v ufw && sudo ufw allow ssh`).
  - Duplicated copies to keep in sync: steps `6.0`-`15.0` exist in the main guide plus
    `lvm-disk-layout.md`, `disk-encryption.md`, `lvm-on-luks.md`, and step `15.0` in four
    `limine-bootloader*.md` files. An edit to one copy must go to all.
- Links: anchors are GitHub heading slugs, so renaming or renumbering a heading breaks them. No
  link checker exists: extract links with `grep -oE '\]\([^)]+\)'`, confirm targets exist, and
  recompute slugs for `#fragments`.
- Keep prose tight: say only what the reader needs to act, one explanation per command, no preamble.
- Placeholders: values to substitute are `<angle-bracketed>` (e.g. `/dev/<your-disk>`); plain
  values like `wlan0` or `vg` are examples or names the guide creates. Don't present realistic
  literals like `/dev/nvme0n1` as values to type, only as `e.g.` examples.
- Everything from `11.0` to `21.0` runs in `arch-chroot`, where systemd isn't PID 1:
  `timedatectl`/`localectl` fail there, so use file-based config (`/etc/localtime`,
  `/etc/locale.conf`) and `systemctl enable` only. `/boot` is FAT32, so `chmod` on it doesn't
  work.
- Keyboard layout (`1.0`) stays the very first step; `12.0` re-offers the layouts for
  `KEYMAP` rather than referring back. `HOOKS` uses `udev`, `keymap`, and
  `consolefont` instead of `systemd`/`sd-vconsole`; `keymap` is what applies a non-US layout at
  the LUKS prompt, so don't re-add it as if missing. The `microcode` hook embeds microcode, so no
  bootloader `initrd`/`module_path` lines for it.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
