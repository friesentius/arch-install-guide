# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Documentation-only repo (no code, CI, or tests). "Correctness" means the steps work on real
  Arch Linux and the Markdown links resolve. Check package/hook facts against a real Arch system
  (`pacman -Si`, `/usr/lib/initcpio/install/`) rather than memory.
- Layout (the captain's chosen structure): `arch-linux-install-guide.md` is one numbered sequence
  of steps (`1.0`-`36.0`). Each step is either a single set of commands or a list of
  `#### Option: <name>` headings with exactly one labeled `(default)`. An option holds only the
  commands that differ for that step; the reader then continues with the next step. Only an
  option too long to sit inline goes in `branches/`, and it must return to the very next step
  (`branches/README.md`'s table lists them; keep it and `README.md` in sync).
- "Fog" rule: a reader never needs to remember an earlier choice. Where an earlier choice changes
  a later step (disk layout at `7.0`-`10.0`, `13.0`-`15.0`; networking at `24.0`), that step is
  itself multi-option, with each option named by what's plainly true of the system ("Encrypted
  disk (LUKS)", "LVM inside LUKS", "Wi-Fi with NetworkManager"). No "if you took", "back in step
  N", reminders, or lists of steps to watch for. Prefer neutralizing a choice's later effect over
  adding options downstream (e.g. the doas option symlinks `sudo`; `$OPTS` at `15.0` feeds
  either bootloader; `command -v ufw && sudo ufw allow ssh` adapts by itself).
- Links: anchors are GitHub heading slugs, so renaming or renumbering a heading breaks them. No
  link checker exists: extract links with `grep -oE '\]\([^)]+\)'`, confirm targets exist, and
  recompute slugs for `#fragments`.
- Keep prose tight: say only what the reader needs to act, one explanation per command, no preamble.
- Placeholders: values to substitute are `<angle-bracketed>` (e.g. `/dev/<your-disk>`); plain
  values like `wlan0` or `vg` are examples or names the guide creates. Don't present realistic
  literals like `/dev/nvme0n1` as values to type, only as `e.g.` examples.
- Everything from `11.0` to `21.0` runs in `arch-chroot` (bash), where systemd isn't PID 1:
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
