# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Documentation-only repo (no code, CI, or tests). "Correctness" means the steps work on real
  Arch Linux and the Markdown links resolve. Check package/hook facts against a real Arch system
  (`pacman -Si`, `/usr/lib/initcpio/install/`) rather than memory.
- Layout (the captain approved this flow): `arch-linux-install-guide.md` holds only the numbered
  steps (`1.0`-`34.0`), with nothing for a reader to skip over. Two step shapes: a **choice** step
  lists links to option docs, one labeled `(default)`; an **optional** step links to a detour doc
  the reader can skip. Every option/detour is its own doc in `options/`, named `NN-<name>.md` after
  its step, and ends with exactly one `Continue at [N.0 Title](../arch-linux-install-guide.md#...)`
  line pointing at the very next step. `options/README.md` indexes them; keep it and `README.md`
  in sync.
- "Fog" rule: a reader never needs to remember an earlier choice, so no later step may depend on
  one. Neutralize instead of adding downstream options: each `06-disk-*.md` does everything
  layout-specific (format, mount, active swap and `/tmp` mount options that `genfstab` then
  records, plus `/mnt/etc/mkinitcpio.conf.d/disk.conf` hooks and `/mnt/etc/kernel/cmdline` boot
  options, written before `pacstrap`); `lvm2` is always installed; both bootloader docs read
  `/etc/kernel/cmdline`; the doas doc symlinks `sudo`; NetworkManager is a post-install detour so
  base networking is always iwd; `command -v ufw && sudo ufw allow ssh` adapts by itself.
- Links: anchors are GitHub heading slugs, so renaming or renumbering a heading breaks them. No
  link checker exists: extract links with `grep -oE '\]\([^)]+\)'`, confirm targets exist, and
  recompute slugs for `#fragments`.
- Keep prose tight: say only what the reader needs to act, one explanation per command, no preamble.
- Placeholders: values to substitute are `<angle-bracketed>` (e.g. `/dev/<your-disk>`); plain
  values like `wlan0` or `vg` are examples or names the guide creates. Don't present realistic
  literals like `/dev/nvme0n1` as values to type, only as `e.g.` examples.
- Everything from `9.0` to `18.0` runs in `arch-chroot` (bash), where systemd isn't PID 1:
  `timedatectl`/`localectl` fail there, so use file-based config (`/etc/localtime`,
  `/etc/locale.conf`) and `systemctl enable` only. `/boot` is FAT32, so `chmod` on it doesn't
  work.
- Keyboard layout (`1.0`) stays the very first step; `10.0` re-offers the layouts for
  `KEYMAP` rather than referring back. `HOOKS` uses `udev`, `keymap`, and
  `consolefont` instead of `systemd`/`sd-vconsole`; `keymap` is what applies a non-US layout at
  the LUKS prompt, so don't re-add it as if missing. The `microcode` hook embeds microcode, so no
  bootloader `initrd`/`module_path` lines for it.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
