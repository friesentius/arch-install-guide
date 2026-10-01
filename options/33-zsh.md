# 33.0 Shell: Switch to zsh

zsh is bash-compatible, with a large ecosystem of frameworks such as Oh My Zsh.
```shell
sudo pacman -S zsh
chsh -s /usr/bin/zsh
```
`chsh` asks for your password; log out and back in to start using zsh. On its first launch, with
no `~/.zshrc` yet, zsh offers a setup wizard; follow it, or press `0` to create an empty
`~/.zshrc`.

#### Customize ~/.zshrc:
```shell
nano ~/.zshrc
```
```zsh
alias ll='ls -lah'
export EDITOR=nano
autoload -Uz compinit && compinit  # enable zsh's richer tab-completion
```
Changes apply to new shells: open a new terminal, or run `source ~/.zshrc`.

Continue at [34.0 Dotfiles](../arch-linux-install-guide.md#340-dotfiles).
