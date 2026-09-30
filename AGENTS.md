# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- This is a documentation-only repo (no code, no CI, no test suite). "Correctness" here means the
  install steps actually work on real Arch Linux and the Markdown renders/links cleanly.
- Layout: `arch-linux-install-guide.md` is **one main guide with a single globally-numbered
  sequence of steps** (`1.0` through `30.0`, not reset per section) spanning pre-install, disk
  layout, base install, system configuration, bootloader, finalize/reboot, verify, and
  post-install configuration - there is no separate "core guide vs. appendices" split anymore.
  `branches/` holds every optional/alternative path (LVM disk layout, disk encryption via LUKS,
  Limine bootloader, alternate login shell, alternate privilege escalation via opendoas,
  graphics/AUR extras, firewall, SSH hardening, update hygiene, system config, shell/dotfiles
  config) as a branch file, each forking from and rejoining a specific numbered step of the main
  guide - see `branches/README.md`'s table for the exact fork/rejoin step of each one. `README.md`
  and `branches/README.md` are the entry points - keep both in sync with the actual file set and
  step numbers when adding/removing/renaming guide files or steps.
- Post-install steps are real numbered steps (`25.0`-`30.0`), not a bullet list at the end: each
  one states a default (usually "skip") inline in the main guide and links to its branch file for
  the alternative, same as any pre-install fork point. `disk-encryption.md` is the one branch that
  isn't post-install - like `lvm-disk-layout.md`, it's a pre-install decision made at step `5.0`
  Choose Your Disk Layout, since it changes the disk-layout/initramfs/bootloader steps that follow.
- Click-through convention: every fork point in the main guide carries an inline "Want X instead?
  -> branches/y.md" link, and every branch ends with a "Continue in the main guide" link back to
  the specific next step **by number and heading anchor** (e.g.
  `arch-linux-install-guide.md#160-enable-networking-services`). Keep both ends of this pathway in
  sync when adding, removing, or reordering steps - anchors are GitHub's auto-generated heading
  slugs (lowercase, punctuation stripped, spaces to hyphens), not something declared in the file,
  so renumbering a step's heading breaks any anchor link that pointed at it, and renumbering one
  step means re-checking every branch file that references that step by number, not just the
  anchor. `branches/README.md`'s table is the single source of truth for which step each branch
  forks from and rejoins - update it whenever a fork/rejoin point changes.
- Self-containment rule (deliberate design choice, not an oversight if you see repeated content):
  no step may require the reader to recall an earlier choice or click into another file to learn
  what to type - "the only context they should need is in the paragraph they're reading." When an
  earlier fork changes a later step's content (disk layout changing the `pacstrap` package list at
  `9.0`, swap commands at `14.0`, initramfs `HOOKS` at `15.0`, and bootloader `root=`/`cmdline` at
  `20.0`; privilege escalation changing `sudo` to `doas` in post-install branches; shell changing
  heredoc support), that later step lists every variant in full, labeled by which earlier choice it
  matches (see `9.0`/`14.0`/`15.0`/`20.0` in the main guide, and `limine-bootloader.md`'s cmdline
  section, for the pattern) - never a bare "if you took branch X, see that file" pointer. Disk-layout
  branches (`lvm-disk-layout.md`, `disk-encryption.md`) therefore rejoin the main guide exactly
  once, at `9.0`, instead of bouncing the reader back and forth at `14.0`/`15.0`/`20.0`. A branch
  keeps its own dedicated file only for genuinely distinct, multi-command procedures (partitioning,
  a different bootloader's whole config file, a privilege-escalation tool's setup); short
  branch-specific values that a later step needs are inlined on the main guide instead of living
  only in the branch. Post-install branches that use `sudo` state the `doas` substitution inline
  near the top of the file rather than assuming `sudo` is installed.
- Placeholder convention: a value the reader must substitute for their own system (device paths,
  hostname, username, SSID, etc.) is written as `<angle-bracket-name>`, e.g. `/dev/<your-disk>`,
  `<your-username>`. Plain unbracketed example values (like `wlan0` or `vg`) are illustrative
  only. Keep this convention consistent when editing; don't reintroduce realistic-looking literal
  values (e.g. `/dev/nvme0n1`) as if they were something to type verbatim.
- No automated link or anchor checker exists; when adding/moving/renaming a `.md` file or
  changing a heading, manually verify every relative Markdown link (and any `#anchor` fragment)
  between README.md, the main guide, and branches/ still resolves (e.g.
  `grep -oE '\]\([^)]+\)'` per file, check the file part exists relative to that file's dir, and
  recompute the anchor slug for anything with a `#fragment`).
- Keyboard layout: the main guide's live-ISO keyboard step (`arch-linux-install-guide.md`, step
  `1.0`) is the very first command in Pre-Installation, ahead of everything else - don't let it
  drift later in the sequence. The main guide's initramfs `HOOKS` line already carries `keyboard
  keymap consolefont` (the non-systemd equivalents of the default `HOOKS`' `systemd`/`sd-vconsole`,
  since this guide uses `udev` not `systemd` in `HOOKS`); `keymap` reads `/etc/vconsole.conf`'s
  `KEYMAP` into the initramfs, which is what makes a non-US layout apply at the LUKS passphrase
  prompt in `branches/disk-encryption.md` - don't re-add it as if it were missing.

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
