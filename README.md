# Corbin's Auto-Rice Bootstrapping Script (CARBS)

## Installation

On an Arch-based distribution, as root, run the following:

```
curl -LO raw.githubusercontent.com/corbin-zip/CARBS/master/carbs.sh
sh carbs.sh
```

That's it.

## What is CARBS?

CARBS is a script that installs and configures a minimal, terminal-and-vim based Arch Linux environment. It's a fork of Luke Smith's LARBS with my own dotfiles, my own program list, and my own builds of the suckless tools (dwm, dwmblocks, dmenu, st).

It can be run on a fresh install of Arch or Artix Linux and leaves you with a fully configured diving-board for work or further customization.

## CARBS as a fork

The biggest difference is in what the script deploys. [The dotfiles](https://github.com/corbin-zip/dotfiles) have grown well past their voidrice roots: additional scripts (statusbar modules, sync cron jobs, dmenu utilities), a streamlined theming pipeline, and various bugfixes. The suckless forks ([dwm](https://github.com/corbin-zip/dwm), [dwmblocks](https://github.com/corbin-zip/dwmblocks), [dmenu](https://github.com/corbin-zip/dmenu), [st](https://github.com/corbin-zip/st)) are patched up to their current upstream versions rather than frozen in the past, with further enhancements, bugfixes, and performance work described in each repo's README.

The script itself also differs --

- Dotfiles are deployed with GNU `stow` instead of copied into `$HOME`. The repo stays a normal working clone in `~/.local/src/dotfiles` and your home directory is symlinks into it, so updating or committing config changes is just git.
- A second, private dotfiles repo can be stowed on top of the public one with `-s`, for the configs that don't belong on GitHub.
- Test mode (`-t`) validates every entry in the program list against the official repos, the AUR, and git without installing anything, so a renamed or dropped package is caught before it dies an hour into an install.
- The script is plain POSIX `sh` and written to be idempotent: re-running it on an already-installed system is safe.
- A pile of bugfixes over upstream, many hunted down with Claude Code running Fable 5. See the commit history.

## Customization

By default, CARBS installs the programs listed in [progs.csv](progs.csv) and [my dotfiles](https://github.com/corbin-zip/dotfiles). You can change either by editing the variables at the top of the script, or by passing options:

- `-r`: custom dotfiles repository (URL)
- `-s`: private dotfiles repository (URL), stowed on top of the main one
- `-p`: custom programs list (local file or URL)
- `-a`: custom AUR helper (must be able to install with `-S`, unless you change the relevant line in the script)
- `-t`: test mode. Checks that every program in the list is still available from the official repos, the AUR, or git, without installing anything. Doesn't require root.

### The `progs.csv` list

CARBS parses the given programs list and installs every program in it. The file must be a three column `.csv`.

The first column is a tag that determines how the program is installed: "" (blank) for the main repository, `A` for the AUR, or `G` for a git repository that is meant to be `make && sudo make install`ed.

The second column is the name of the program in the repository (or the link to the git repository), and the third column is a description: a verb phrase that CARBS prints as part of a sentence during installation. The descriptions also double as documentation for anyone reading the CSV who wants to install things manually.

For `G` entries, the URL may end in `#branch` to build a specific branch instead of the repo's default (e.g. `https://github.com/corbin-zip/st.git#reflow`). Test mode verifies that the branch exists.

Programs are installed from top to bottom, so if one program depends on another, order them accordingly.

If a description contains commas, wrap the whole description in double quotes so it parses correctly.

### Choosing an st build

[My st fork](https://github.com/corbin-zip/st) has two long-lived branches: `master`, with the classic scrollback patch & related fixes (see below), and `reflow`, where scrollback rewraps when the window is resized. CARBS builds `master` by default. To get the reflow build instead, edit the st line in `progs.csv` to point at the branch:

```
G,https://github.com/corbin-zip/st.git#reflow,"is my custom build of suckless's terminal emulator (reflow branch)."
```

A commented-out copy of that line already sits next to the default in `progs.csv`, so you can just swap which one is active. The st repo's README covers the differences between the two builds in more detail.

#### classic scrollback

Traditionally with this patch if you were to resize the window small and then large again, you would lose text that was hidden during the resize; this has been resolved on my `master` st branch, which I feel makes classic scrollback still a perfectly viable option over the more "modern" `reflow`.

### The script itself

The script is divided into functions for easier readability and troubleshooting. Most of it should be self-explanatory.

The main work is done by the `installationloop` function, which iterates through the programs file and picks the install method based on each program's tag. Adding new tags and install methods is easy.

Dotfiles are cloned into `~/.local/src` and symlinked into the home directory with GNU `stow`.

Programs from the AUR can only be built by a non-root user, so during installation CARBS temporarily allows the new user to use `sudo` without a password (via a file in `/etc/sudoers.d`, so the user isn't prompted over and over). At the end of installation that file is removed, and the user is left with normal password-protected `sudo` plus a handful of basic commands that work without a password (`shutdown`, `reboot`, `pacman -Syu` and the like).
