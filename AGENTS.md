# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- This is a documentation-only repo (no code, no CI, no test suite). "Correctness" here means the
  install steps actually work on real Arch Linux and the Markdown renders/links cleanly.
- Layout: `arch-linux-install-guide.md` is the core guide (basic UEFI install: single EFI +
  ext4 root partition, swapfile, sudo, systemd-boot), ending in a "Next Steps" section.
  `appendices/` holds optional/alternative topics (LVM disk layout, disk encryption via LUKS,
  Limine bootloader, alternate login shell, alternate privilege escalation via opendoas,
  graphics/AUR extras, firewall, SSH hardening, update hygiene, system config, shell/dotfiles
  config) that each explicitly say which core sections they replace or add to. `README.md` and
  `appendices/README.md` are the entry points - keep both in sync with the actual file set when
  adding/removing/renaming guide files.
- Pre-install vs. post-install appendices: most appendices (firewall, ssh-hardening,
  automatic-updates, system-config, shell-config) are post-install add-ons, linked from the core
  guide's end-of-guide "Next Steps" section. `disk-encryption.md` is the odd one out - like
  `lvm-disk-layout.md`, it's a pre-install decision that changes the disk-layout/initramfs/
  bootloader steps, so it's linked from a callout at the top of the core guide's disk-layout
  step instead of from Next Steps, where it would no longer be actionable.
- Click-through convention: every genuine fork point in the core guide (a step an appendix
  replaces or inserts after) carries an inline "Want X instead? -> appendices/y.md" link, and
  every appendix ends with a "Continue in the core guide" link back to the specific next core
  step (by heading anchor, e.g. `arch-linux-install-guide.md#70-enable-networking-services`).
  Keep both ends of this pathway in sync when adding, removing, or reordering steps/headings -
  anchors are GitHub's auto-generated heading slugs (lowercase, punctuation stripped, spaces to
  hyphens), not something declared in the file, so renumbering a step's heading breaks any anchor
  link that pointed at it.
- Placeholder convention: a value the reader must substitute for their own system (device paths,
  hostname, username, SSID, etc.) is written as `<angle-bracket-name>`, e.g. `/dev/<your-disk>`,
  `<your-username>`. Plain unbracketed example values (like `wlan0` or `vg`) are illustrative
  only. Keep this convention consistent when editing; don't reintroduce realistic-looking literal
  values (e.g. `/dev/nvme0n1`) as if they were something to type verbatim.
- No automated link or anchor checker exists; when adding/moving/renaming a `.md` file or
  changing a heading, manually verify every relative Markdown link (and any `#anchor` fragment)
  between README.md, the core guide, and appendices/ still resolves (e.g.
  `grep -oE '\]\([^)]+\)'` per file, check the file part exists relative to that file's dir, and
  recompute the anchor slug for anything with a `#fragment`).

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
