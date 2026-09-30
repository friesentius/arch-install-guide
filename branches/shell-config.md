# Branch: Customize bash

The [main guide](../arch-linux-install-guide.md)'s 30.0 Shell Configuration, for customizing
bash. Run it from your regular user.

## Edit ~/.bashrc
```shell
nano ~/.bashrc
```
bash runs `~/.bashrc` every time it starts interactively; aliases, prompt settings, and
environment variables go there. Your account already has one from Arch's defaults. Common
additions:
```bash
alias ll='ls -lah'
export EDITOR=nano
PS1='[\u@\h \W]\$ '  # customize the prompt
```
Changes apply to new shells: open a new terminal, or run `source ~/.bashrc`.

## Continue in the main guide
Continue at [31.0 Dotfiles](../arch-linux-install-guide.md#310-dotfiles).
