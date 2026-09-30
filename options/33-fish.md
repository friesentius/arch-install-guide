# 33.0 Shell: Switch to fish

fish isn't bash-compatible, but has autosuggestions, syntax highlighting, and tab-completion with
no configuration.
```shell
sudo pacman -S fish
chsh -s /usr/bin/fish
```
`chsh` asks for your password; log out and back in to start using fish.

#### Customize ~/.config/fish/config.fish:
```shell
mkdir -p ~/.config/fish
nano ~/.config/fish/config.fish
```
```fish
alias ll 'ls -lah'
set -gx EDITOR nano
```
Changes apply to new shells: open a new terminal, or run `source ~/.config/fish/config.fish`.

Continue at [34.0 Dotfiles](../arch-linux-install-guide.md#340-dotfiles).
