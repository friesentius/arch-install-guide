# Branch: Alternate Login Shell (zsh / fish)

This branch is an addition to the [main guide](../arch-linux-install-guide.md)'s 18.0 Add User
step: it installs and sets an alternative login shell for your new user account instead of the
default bash. It doesn't replace anything in the main guide - it's an optional extra step in
between 18.0 Add User and 19.0 Configure Privilege Escalation.

**Use this branch if:** you'd rather use zsh or fish as your everyday shell instead of bash.
This guide's default steps use bash (`-s /bin/bash` in `useradd`) since it's always present and
needs no extra package; `zsh` and `fish` are the two most common alternatives, both covered
below.

Run this from inside the chroot, right after creating your user in the main guide's 18.0 Add
User step - no need to wait for your first boot.

## 1.0 Install Your Shell of Choice
Pick one (or both, if you want to leave bash available and switch later):

#### zsh:
```shell
pacman -S zsh
```

#### fish:
```shell
pacman -S fish
```
`zsh` is a bash-compatible shell with a large ecosystem of frameworks (Oh My Zsh, etc.) and is
the closer drop-in if you're used to bash. `fish` isn't POSIX/bash-compatible, but is friendly
out of the box - autosuggestions, syntax highlighting, and sane defaults with no configuration
needed.

## 2.0 Set It as Your User's Login Shell
```shell
chsh -s /usr/bin/zsh <your-username>  # e.g. archie
```
or
```shell
chsh -s /usr/bin/fish <your-username>  # e.g. archie
```
`chsh` changes the shell your login prompt starts in, recorded in `/etc/passwd`. Use whichever
line matches the package you installed above, and replace `<your-username>` with the username
you created in the main guide's 18.0 Add User step. If you're not sure the binary is really at
`/usr/bin/...` on your system, check first with `which zsh` or `which fish`.

## 3.0 (Optional) Starter Config
A brand-new shell has no config file yet, so its first launch may prompt you about it (fish
offers to open its web config; zsh offers to run a first-time setup wizard). Either is fine to
accept, or create an empty config yourself to skip the prompt:

```shell
# zsh
touch /home/<your-username>/.zshrc  # e.g. /home/archie/.zshrc
```

```shell
# fish
mkdir -p /home/<your-username>/.config/fish             # e.g. /home/archie/.config/fish
touch /home/<your-username>/.config/fish/config.fish     # e.g. /home/archie/.config/fish/config.fish
```
This branch doesn't cover shell theming/configuration beyond this - see
[branches/shell-config.md](shell-config.md) for that once your system is installed. Personal
dotfiles beyond a config file are a taste rabbit hole well outside the scope of getting a
bootable system.

## Continue in the main guide
Once your shell is set, continue with the main guide's
[19.0 Configure Privilege Escalation](../arch-linux-install-guide.md#190-configure-privilege-escalation-sudo).
