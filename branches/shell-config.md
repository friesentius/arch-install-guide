# Branch: Shell Configuration and Dotfiles Management

The [main guide](../arch-linux-install-guide.md)'s 30.0 Shell Configuration and Dotfiles, the
last step: customizing your shell's config file, and keeping config files in version control.

## Your Shell's Config File

Your shell runs its config file every time it starts interactively; aliases, prompt settings,
and environment variables go there. Check your shell with `echo $SHELL` and use the matching
section below. For `EDITOR`, use the editor `pacman -Q nano neovim vim 2>/dev/null` shows.

#### bash: `~/.bashrc`
```shell
nano ~/.bashrc
```
```bash
alias ll='ls -lah'
export EDITOR=nano  # replace nano with whichever editor the check above showed
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
fish uses its own syntax, as shown, and already has tab-completion without any config.

Changes apply to new shells. Open a new terminal, or reload the file in the current one (e.g.
`source ~/.bashrc`).

## Managing Dotfiles Long-Term

Keeping your config files ("dotfiles": `.bashrc`, `.gitconfig`, `.vimrc`, and so on) in version
control lets you restore them after a reinstall or copy them to a new machine. Two common
approaches:

- **A bare git repository:** `git init --bare ~/.dotfiles`, plus a `dotfiles` alias for
  `git --git-dir=$HOME/.dotfiles/ --work-tree=$HOME`. You then `add`/`commit`/`push` files
  straight from `$HOME`, with no symlinks. Search "dotfiles bare git repository" for a full
  walkthrough. Simplest for one set of configs.
- **A dotfiles manager such as [chezmoi](https://www.chezmoi.io/):** templates handle differences
  between machines (hostnames, secrets, work vs. personal). More to learn, but pays off across
  several machines.

## Continue
This is the end of the path. The [branch index](README.md) lists every other optional step if you
want to revisit one.
