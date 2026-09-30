# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- Documentation-only repo (no code, CI, or tests). "Correctness" means the steps work on real
  Arch Linux and the Markdown links resolve. Check package/hook facts against a real Arch system
  (`pacman -Si`, `/usr/lib/initcpio/install/`) rather than memory.
- Layout: `arch-linux-install-guide.md` is one path with globally numbered steps (`1.0`-`30.0`);
  post-install steps (`25.0`-`30.0`) are numbered steps too, each defaulting to "skip".
  `branches/` holds every alternative, each forking from and rejoining a numbered step.
  `branches/README.md`'s table is the source of truth for fork/rejoin points. Keep it and
  `README.md` in sync with the file set and step numbers.
- Links: every fork point links to its branch, and every branch ends with a link back to its
  rejoin step by heading anchor (e.g. `arch-linux-install-guide.md#160-enable-networking-services`).
  Anchors are GitHub's heading slugs, so renaming or renumbering a heading breaks them; after any
  heading change, grep every file for the old number and anchor. There's no link checker: extract
  links with `grep -oE '\]\([^)]+\)'`, confirm each target file exists, and recompute slugs for
  `#fragments`.
- "Fog" rule (deliberate, not an oversight): each paragraph must be actionable on its own.
  - Never depend on memory of an earlier choice ("back in step N", "the branch you took"). When
    an earlier fork changes a later step (disk layout at `9.0`/`10.0`/`14.0`/`15.0`/`20.0` and
    in `limine-bootloader.md`; `sudo` vs `doas`; keymap), that step lists every variant, labeled.
  - Open such a step with a live check instead: `lsblk -f` (`crypto_LUKS`/`LVM2_member`) for disk
    layout; `systemctl is-enabled sshd` for SSH; `which sudo doas` for privilege escalation;
    `echo $SHELL` for the shell; `pacman -Q nano neovim vim` for the editor; typing `@`/`:` for
    the keymap (`loadkeys` leaves nothing to query). Reuse these rather than inventing new ones,
    and prefer shell-agnostic commands (e.g. `printf ... | sudo tee`, not heredocs, which fish
    lacks) over per-shell variants.
  - Disk-layout branches rejoin once, at `9.0`. A branch gets its own file only for a distinct
    multi-command procedure; short branch-specific values a later step needs are inlined there.
- Keep prose tight: say only what the reader needs to act, one explanation per command, no preamble.
- Placeholders: values to substitute are `<angle-bracketed>` (e.g. `/dev/<your-disk>`); plain
  values like `wlan0` or `vg` are examples or names the guide creates. Don't present realistic
  literals like `/dev/nvme0n1` as values to type, only as `e.g.` examples.
- Everything from `11.0` to `21.0` runs in `arch-chroot`, where systemd isn't PID 1:
  `timedatectl`/`localectl` fail there, so use file-based config (`/etc/localtime`,
  `/etc/locale.conf`) and `systemctl enable` only. `/boot` is FAT32, so `chmod` on it doesn't
  work.
- Keyboard layout (`1.0`) stays the very first step. `HOOKS` uses `udev`, `keymap`, and
  `consolefont` instead of `systemd`/`sd-vconsole`; `keymap` is what applies a non-US layout at
  the LUKS prompt, so don't re-add it as if missing. The `microcode` hook embeds microcode, so no
  bootloader `initrd`/`module_path` lines for it.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
