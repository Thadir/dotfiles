# Setup — a clean machine

Getting these dotfiles onto a machine that has never had them.
To update a machine that already has them, see [`update.md`](update.md).

<br>

Works on **macOS (Apple Silicon)**, **Linux**, and **Windows via WSL2**. chezmoi
is cross-platform; the macOS-only bits (Homebrew bundle) are guarded and skip
themselves elsewhere.

<br>

---

<br>

## 1. Prerequisites

<br>

### macOS

Nothing. `git`, `curl` and `zsh` ship with the OS. The bootstrap installs
Homebrew and everything in the Brewfile.

<br>

### Linux

```sh
sudo apt install -y git curl zsh        # Debian / Ubuntu
# sudo dnf install -y git curl zsh      # Fedora
# sudo pacman -S --needed git curl zsh  # Arch
```

Install the CLI tools the prompt and aliases expect
(`starship`, `vivid`, plus `coreutils` if `ls` isn't already GNU) with your
package manager or [Homebrew on Linux](https://docs.brew.sh/Homebrew-on-Linux).

<br>

### Windows (WSL2)

1. Install WSL2 with a distro: `wsl --install` in an admin PowerShell, reboot,
   finish the Linux user setup.
2. **Inside WSL**, do the Linux prerequisites above.
3. **On Windows** (not WSL): install a **Nerd Font** — "JetBrainsMono Nerd
   Font" — and set it as the font for the WSL profile in Windows Terminal
   (Settings → your distro → Appearance → Font face). This is what renders the
   prompt's Powerline and icon glyphs; without it they show as blank boxes.

<br>

---

<br>

## 2. SSH keys and `~/.ssh/config`

<br>

The git identities route `git@github.com:` URLs through **host aliases** so each
account uses its own key. Create the keys and aliases *before* running chezmoi
so the values you enter at the prompts are real.

<br>

For each account you'll use (personal, employer, any EMU or customer account):

```sh
ssh-keygen -t ed25519 -C "you@example.com" -f ~/.ssh/id_ed25519_personal
# add ~/.ssh/id_ed25519_personal.pub to that GitHub account
```

```sshconfig
# ~/.ssh/config
Host github.com-personal
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_personal
    IdentitiesOnly yes

Host github.com-work
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_work
    IdentitiesOnly yes
```

Add one `Host` block per account. The alias names are yours to choose — you'll
type them at the chezmoi prompts.

<br>

---

<br>

## 3. Run the bootstrap

<br>

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply Thadir
```

This installs chezmoi, clones the repo to `~/.local/share/chezmoi`, runs the
prompts, and applies everything.

<br>

### The prompts

| Prompt | What it's for |
| --- | --- |
| Personal name / email / SSH host alias | The **default** identity — used for every repo not matched below. |
| Employer name / email / SSH host alias | Your main corporate identity. |
| EMU name / email / SSH host alias | A separate Enterprise Managed User account, if your organisation issues one for certain repos. Leave the email blank if you don't have one. |
| Projects root | Where your repos live (default `~/projects`). |
| Employer dirs / EMU dirs | Comma-separated subdirs of the projects root that use each work account. Everything else uses the personal identity. |

Answers are written to `~/.config/chezmoi/chezmoi.toml`. That file is **local
only** — it is never committed and holds the only copy of your names, emails and
host aliases.

<br>

---

<br>

## 4. Store secrets

<br>

The shell pulls a GitHub token from the OS keychain at startup. Store it once —
the value is prompted and never echoed:

```sh
# macOS
security add-generic-password -a "$USER" -s github_token -w

# Linux / WSL (whichever is installed)
pass insert cli/github_token
secret-tool store --label='github token' service github_token
```

Until it's stored, every shell prints one yellow reminder line.

<br>

---

<br>

## 5. Finish

<br>

```sh
chsh -s "$(command -v zsh)"   # make zsh your login shell (if it isn't)
exec zsh                      # or just open a new terminal
```

<br>

---

<br>

## Adding a customer account

<br>

For a customer-issued account, add a block to `~/.config/chezmoi/chezmoi.toml`:

```toml
[[data.customer]]
    slug     = "acme"                   # directory name + the .inc file suffix
    email    = "you@acme-external.com"
    # optional:
    name     = "Your Name"              # defaults to the personal name
    dir      = "~/work/acme"            # defaults to <projects_root>/<slug>
    ssh_host = "github.com-acme"        # adds an insteadOf route; omit if not needed
```

```sh
chezmoi apply
```

That adds the `includeIf` to `~/.gitconfig` and writes
`~/.config/git/customer-acme.inc`. Remove the block and re-apply to delete it.
Blocks survive `chezmoi init`.

<br>

---

<br>

## Verify

<br>

```sh
git -C <a personal repo>  config user.email   # -> personal
git -C <a work repo>       config user.email   # -> employer / EMU / customer
git config --show-origin user.email            # shows which .inc supplied it
```
