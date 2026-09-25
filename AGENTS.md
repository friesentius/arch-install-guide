# Project agent memory

This file is the project's committed home for project-intrinsic agent knowledge: build, test, release, architecture, and sharp-edge notes that should travel with the code.

- This is a documentation-only repo (no code, no CI, no test suite). "Correctness" here means the
  install steps actually work on real Arch Linux and the Markdown renders/links cleanly.
- Layout: `arch-linux-install-guide.md` is the core guide (basic UEFI install: single EFI +
  ext4 root partition, swapfile, systemd-boot). `appendices/` holds optional/alternative topics
  (LVM disk layout, Limine bootloader, graphics/AUR extras) that each explicitly say which core
  sections they replace or add to. `README.md` and `appendices/README.md` are the entry points -
  keep both in sync with the actual file set when adding/removing/renaming guide files.
- Placeholder convention: a value the reader must substitute for their own system (device paths,
  hostname, username, SSID, etc.) is written as `<angle-bracket-name>`, e.g. `/dev/<your-disk>`,
  `<your-username>`. Plain unbracketed example values (like `wlan0` or `vg`) are illustrative
  only. Keep this convention consistent when editing; don't reintroduce realistic-looking literal
  values (e.g. `/dev/nvme0n1`) as if they were something to type verbatim.
- No automated link checker exists; when adding/moving/renaming a `.md` file, manually verify
  every relative Markdown link between README.md, the core guide, and appendices/ still resolves
  (e.g. `grep -oE '\]\([^)]+\)'` per file and check the target exists relative to that file's dir).

## Maintaining this file

Keep this file for knowledge useful to almost every future agent session in this project.
Do not repeat what the codebase already shows; point to the authoritative file or command instead.
Prefer rewriting or pruning existing entries over appending new ones.
When updating this file, preserve this bar for all agents and keep entries concise.
