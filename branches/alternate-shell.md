# Branch: Alternate Login Shell (zsh / fish)

An addition to the [main guide](../arch-linux-install-guide.md)'s 18.0 Add User: run it inside
the chroot right after creating your user, to use zsh or fish instead of bash.

## 1.0 Install Your Shell of Choice

#### zsh:
```shell
pacman -S zsh
```

#### fish:
```shell
pacman -S fish
```
`zsh` is bash-compatible, with a large ecosystem of frameworks such as Oh My Zsh. `fish` isn't
bash-compatible, but has autosuggestions, syntax highlighting, and good defaults with no
configuration.

## 2.0 Set It as Your User's Login Shell
```shell
chsh -s /usr/bin/zsh <your-username>  # e.g. archie
```
or
```shell
chsh -s /usr/bin/fish <your-username>  # e.g. archie
```
Use the line matching the shell you installed. On zsh's first launch, with no `~/.zshrc` yet, it
offers a setup wizard; follow it, or press `0` to create an empty `~/.zshrc` and skip it.
Customizing the shell comes at the end of the install, in 30.0 Shell Configuration and Dotfiles.

## Continue in the main guide
Continue at [19.0 Configure Privilege Escalation](../arch-linux-install-guide.md#190-configure-privilege-escalation-sudo).
