# 34.0 Dotfiles

Keeping your config files ("dotfiles": `.bashrc`, `.gitconfig`, `.vimrc`, and so on) in version
control lets you restore them after a reinstall or copy them to a new machine. Two common
approaches:

- **A bare git repository:** install git (`sudo pacman -S --needed git`), then
  `git init --bare ~/.dotfiles`, plus a `dotfiles` alias for
  `git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME`. You then `add`/`commit`/`push` files
  straight from `$HOME`, with no symlinks. Search "dotfiles bare git repository" for a full
  walkthrough. Simplest for one set of configs.
- **A dotfiles manager such as [chezmoi](https://www.chezmoi.io/):** templates handle differences
  between machines (hostnames, secrets, work vs. personal). More to learn, but pays off across
  several machines.

Continue at [You're Done](../arch-linux-install-guide.md#youre-done).
