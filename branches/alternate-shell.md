# Branch: Alternate Login Shell (zsh / fish)

The [main guide](../arch-linux-install-guide.md)'s 30.0 Shell Configuration, switching your
login shell from bash to zsh or fish. Run it from your regular user.

`zsh` is bash-compatible, with a large ecosystem of frameworks such as Oh My Zsh. `fish` isn't
bash-compatible, but has autosuggestions, syntax highlighting, and good defaults with no
configuration. Pick one: [zsh](#zsh) or [fish](#fish).

## zsh

#### Install zsh and make it your login shell:
```shell
sudo pacman -S zsh
chsh -s /usr/bin/zsh
```
`chsh` asks for your password. Log out and back in to start using zsh. On its first launch, with
no `~/.zshrc` yet, zsh offers a setup wizard; follow it, or press `0` to create an empty
`~/.zshrc` and skip it.

#### Customize ~/.zshrc:
```shell
nano ~/.zshrc
```
```zsh
alias ll='ls -lah'
export EDITOR=nano
autoload -Uz compinit && compinit  # enable zsh's richer tab-completion
```
Aliases, environment variables, and shell options go here; zsh reads it every time it starts
interactively. Changes apply to new shells: open a new terminal, or run `source ~/.zshrc`.

#### Continue in the main guide
Continue at [31.0 Dotfiles](../arch-linux-install-guide.md#310-dotfiles).

## fish

#### Install fish and make it your login shell:
```shell
sudo pacman -S fish
chsh -s /usr/bin/fish
```
`chsh` asks for your password. Log out and back in to start using fish.

#### Customize ~/.config/fish/config.fish:
```shell
mkdir -p ~/.config/fish
nano ~/.config/fish/config.fish
```
```fish
alias ll 'ls -lah'
set -gx EDITOR nano
```
Aliases and environment variables go here, in fish's own syntax as shown; fish already has
tab-completion without any config. Changes apply to new shells: open a new terminal, or run
`source ~/.config/fish/config.fish`.

#### Continue in the main guide
Continue at [31.0 Dotfiles](../arch-linux-install-guide.md#310-dotfiles).
