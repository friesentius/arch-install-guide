# 33.0 Shell: Customize bash

```shell
nano ~/.bashrc
```
bash runs `~/.bashrc` every time it starts interactively; aliases, prompt settings, and
environment variables go there. Common additions:
```bash
alias ll='ls -lah'
export EDITOR=nano
PS1='[\u@\h \W]\$ '  # customize the prompt
```
Changes apply to new shells: open a new terminal, or run `source ~/.bashrc`.

Continue at [34.0 Dotfiles](../arch-linux-install-guide.md#340-dotfiles).
