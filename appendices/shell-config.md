# Appendix: Shell Configuration and Dotfiles Management

This is one of the "Next Steps" from the end of the
[core install guide](../arch-linux-install-guide.md): it doesn't replace anything there. It
covers the basics of customizing your shell's config file, and a pointer toward managing that
(and other) config files long-term. It ties into the
[alternate-shell appendix](alternate-shell.md) - if you switched to zsh or fish there, use the
matching section below; if you kept the core guide's default bash, use the bash section.

## Your Shell's Config File

Whichever shell you're using, its config file runs every time you open a new interactive shell -
it's where aliases, prompt customization, environment variables, and shell options go.

#### bash: `~/.bashrc`
```shell
nano ~/.bashrc
```
A fresh account already has a `.bashrc` from Arch's `/etc/skel` (or an empty one, if you
created it yourself in the [alternate-shell appendix](alternate-shell.md)). Common additions:
```bash
alias ll='ls -lah'
export EDITOR=nano  # match whatever editor you chose in the core guide's Base Installation
PS1='[\u@\h \W]\$ '  # customize the prompt
```

#### zsh: `~/.zshrc`
```shell
nano ~/.zshrc
```
```zsh
alias ll='ls -lah'
export EDITOR=nano
autoload -Uz compinit && compinit  # enable zsh's richer tab-completion
```

#### fish: `~/.config/fish/config.fish`
```shell
nano ~/.config/fish/config.fish
```
```fish
alias ll 'ls -lah'
set -gx EDITOR nano
```
Fish's syntax for aliases/exports differs from bash/zsh, as shown above; fish also ships with
plenty of sane defaults (like tab-completion) that need no config at all.

Whichever file you edit, changes take effect in *new* shells - either open a new terminal, or
re-run the file in your current one (e.g. `source ~/.bashrc`).

## Managing Dotfiles Long-Term

Once you've got more than a couple of tweaked config files (`.bashrc`, `.gitconfig`, `.vimrc`,
and so on - collectively called "dotfiles"), keeping them in version control means you can carry
them to a new machine (like the one you're setting up right now) or recover them after a
reinstall, instead of recreating everything from memory. This is further reading, not something
this guide walks through step by step - two common approaches:

- **A bare git repository:** track your existing home directory in place, with a git repo that
  has no working tree of its own (`git init --bare ~/.dotfiles`, plus a `dotfiles` shell alias
  wrapping `git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME`). No symlinking needed - just
  `git add`/`commit`/`push` the specific files you want tracked, directly from `$HOME`. This
  technique is well documented online (search "dotfiles bare git repository") if you want a full
  write-up.
- **A dedicated dotfiles manager, e.g. [chezmoi](https://www.chezmoi.io/):** treats dotfiles as
  templates, and handles per-machine differences (different hostnames, secrets, work vs.
  personal machines) more gracefully than a bare git repo, at the cost of a bit more setup and
  its own tool-specific concepts to learn.

Either is a reasonable choice: a bare git repo is simpler if you just want one set of configs
tracked; chezmoi pays off more once you're maintaining dotfiles across several different
machines.

## Continue
This appendix has no further steps of its own. Return to the core guide's
[Next Steps](../arch-linux-install-guide.md#next-steps) for the other optional appendices.
