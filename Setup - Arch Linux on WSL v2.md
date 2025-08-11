# Setting up Arch on Wsl

## Install

Use scoop to install

```console
scoop install archwsl
```

## Add First User

Use the `Arch` utility to enter the environment as root the first time. The first
user will be in the sudoer's group so we'll create that and then the user.
Replace `{username}` with the name of the user you're creating.

```console
$ Arch.exe
[root@PC-NAME]# echo "%wheel ALL=(ALL) ALL" > /etc/sudoers.d/wheel
[root@PC-NAME]# useradd -m -G wheel -s /bin/bash {username}
[root@PC-NAME]# passwd {username}
[root@PC-NAME]# exit
```

Then use the `Arch` utility again to set that user as our default login user.

```console
Arch.exe config --default-user {username}
```

## Set up Packaging

Log in now using either `Arch` or by launching wsl with this distro. Next we'll
set up the `pacman` system (which is the built-in package manager for updates
and installations).

```console
[user@PC-NAME ~]$ sudo pacman-key --init
[user@PC-NAME ~]$ sudo pacman-key --populate
[user@PC-NAME ~]$ sudo pacman -Sy archlinux-keyring
[user@PC-NAME ~]$ sudo pacman -Su
```

### Beautify Pacman (optional)

Add some color and fanciness to the package manager interface by editing the configuration.

```console
EDITOR=vim sudoedit /etc/pacman.conf
```

Enable the options 'Color' and 'ILoveCandy' in the `[options]` section.

## Add Basic Utilities

Get an editor added so we can start setting up configuration files. `micro` is
good for basic editing and can be installed directly with `sudo pacman -S micro`.
Up front you'll want some basic utilities to make subsequent steps smoother:

```console
sudo pacman -S git rust npm unzip base-devel openssh keychain pkgfile
```

Many of these are self-explanatory, but briefly:

- `git` : client for obtaining and managing code
- `rust` : compilation environment that some desired utilities are built with
- `npm` : nodejs' default package manager
- `unzip` : zip format handler
- `base-devel` : basic build environment for linux (includes a c compiler)
- `openssh` : security platform for managing remote connections
- `keychain` : integration layer that makes openssh less cumbersome
- `pkgfile` : utility for searching for a package by their contents (files)

### Paru

A package manager that is a bit friendlier than `pacman`. It currently is only
in the AUR (Arch User Repository) so you'll need to retrieve it manually.

```console
git clone https://aur.archlinux.org/paru-bin.git
cd paru
makepkg -si
```

Afterward you can substitute `paru` wherever you see `pacman` (for single packages),
as well as any of the direct invocations of the `makepkg` from AUR. It will refresh
package lists, search the AUR, offer installation options, and request permission
elevation automatically. Make one modification to the presentation of the package
options so that the most relevant to a given search appear nearest the cursor on
the screen by editing `/etc/paru.conf` and uncommenting the `BottomUp` option.

### Neovim

If you want `neovim` use the version manager utility `bob` to handle it for
you with `sudo pacman -S bob` followed by `bob use stable`. Now neovim will
be installed in a managed folder (`~/.local/share/bob/nvim-bin/`) so you'll
want to add it to your path.

I have a configuration of neovim that I maintain based on [LazyVim](https://www.lazyvim.org/).

```console
git clone https://github.com/GunnerGuyven/nvim_config_lazy.git ~/.config/nvim-lazy
```

Subsequently when you run `nvim` make sure that the env var `NVIM_APPNAME` is
set to 'nvim-lazy'.

#### Extra Utilities for Neovim Productivity

```console
sudo pacman -S wget ripgrep fzf fd lazygit luarocks lua51 rust-analyzer php composer
```

For integration with windows' clipboard install `win32yank` from the AUR.

### Rate-mirrors

To keep the package mirrors up to date and snappy, you'll want to run a utility
to scan available repositories from time to time and measure their performance.
You can use `paru` to search and install this utility `rate-mirrors`, or get it
directly from the AUR with the following commands:

```console
git clone https://aur.archlinux.org/rate-mirrors.git
cd rate-mirrors
makepkg -si
```

After installing this utility is run with the following command, at which point
your mirrors have been refreshed:

```console
rate-mirrors arch | sudo tee /etc/pacman.d/mirrorlist
```

### Starship

This is a cross-shell display configuration manager. Any shell you end up using
(bash, powershell, fish, nushell) can be configured in one place to look nice.

```console
sudo pacman -S starship
```

See the docs for the proper configuration for your given [shell](https://starship.rs/guide/#step-2-set-up-your-shell-to-use-starship).
This is unnecessary if using `nushell` because the correct configuration is
specified in that section.

To configure starship itself you will edit/create a file at `~/.config/starship.toml`.
See [official site](https://starship.rs/config/) for configuration options.

### Carapace

This is an argument completion system that integrates well with nushell (below).
Install from the AUR (using `paru` or using `git` from the aur repos).

### Zoxide

A terminal emulator within a terminal much like `tmux`. Get it from the
official 'extras' repository.

### Nushell

```console
$ sudo pacman -S nushell
$ chsh {username}
Changing Shell for {username}
New shell [/bin/bash]: /usr/bin/nu
Shell changed.
```

Use your chosen editor before to configure: `~/.config/nushell/config.nu`.
You should alter this configuration to your taste.

```nu
$env.config.show_banner = false
$env.config.edit_mode = 'vi'

# for neovim integration
let nvim_path = $env.HOME | path join .local share bob nvim-bin
$env.EDITOR = $nvim_path | path join nvim
$env.NVIM_APPNAME = 'nvim-lazy'

$env.PATH ++= [
 # add anything else you want to PATH here
 $nvim_path
]

$env.CARAPACE_BRIDGES = 'zsh,fish,bash,inshellisense'

let autoload = $nu.data-dir | path join vendor autoload
mkdir $autoload
starship init nu | save -f ($autoload | path join starship.nu)
carapace _carapace nushell | save -f ($autoload | path join carapace.nu)
zoxide init nushell | save -f ($autoload | path join zoxide.nu)

( keychain --eval --quiet
  # ssh keys to auto-load go here
) | split row -r ';.*\s*' | parse -r '(.+)="?([^"]+)' | transpose -r | first | load-env
```

To make future changes to this file you can use the shell command `config nu`.
