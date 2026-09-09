# dotfiles

Personal shell + terminal configuration, managed with
[chezmoi](https://chezmoi.io).

Targets **macOS (Apple Silicon)** today; structured to extend to
**WSL2 / Linux** (Homebrew paths and the `gls` fallback are already
conditional).

<br>

---

<br>

## What's here

<br>

| Target | Source | Purpose |
| --- | --- | --- |
| `~/.zshrc` | `dot_zshrc` | Framework + prompt only. Strict load order: brew shellenv → oh-my-zsh → `.zsh_profile` → `.zsh_secrets` → starship. |
| `~/.zsh_profile` | `dot_zsh_profile` | Personal aliases, `LS_COLORS` (via `vivid`), completion styling, PATH extras, and a dependency check that offers to install `brew` + `vivid` + `coreutils`. |
| `~/.zsh_secrets` | `private_dot_zsh_secrets` | Pulls secrets from the OS keychain at startup. **Contains no secret values.** `chmod 600`. |
| `~/.config/starship.toml` | `dot_config/starship.toml` | Prompt config. **Generated — do not hand-edit.** |
| `~/.config/starship-gen.py` | `dot_config/starship-gen.py` | Generates `starship.toml`; injects Nerd Font glyphs via `chr()` because editors drop them. Run it, then `starship print-config 2>&1 \| grep -iE 'warn\|error'`. |
| `~/.gitconfig` | `dot_gitconfig.tmpl` | Work identity by default; `includeIf` switches to the personal account per directory (see “New machine” and `dot_config/git/`). |
| `~/.config/git/{personal,work}.inc` | `*.inc.tmpl` | Per-account identity + an `insteadOf` rule that routes `git@github.com:` through that account's SSH host alias. |
| `~/.config/homebrew/Brewfile` | `dot_config/homebrew/Brewfile` | Every explicitly-installed formula, cask, VS Code extension and global npm package. Regenerate with `brew bundle dump --force --file=~/.config/homebrew/Brewfile`. |
| _(script)_ | `.chezmoiscripts/run_onchange_after_10-darwin-brew-bundle.sh.tmpl` | macOS only. Installs Homebrew if missing, then runs `brew bundle` — re-runs automatically whenever the Brewfile changes. |
| `~/Library/…/ghostty/config.ghostty` | `private_Library/…` | Ghostty terminal: `font-family = "JetBrainsMono Nerd Font Mono"` (must be the exact family name the terminal exposes), `grapheme-width-method = legacy`, Catppuccin auto light/dark. |

<br>

Identity/email values are **not** in this repo — `chezmoi init` prompts for
them (`.chezmoi.toml.tmpl`) and writes them to `~/.config/chezmoi/chezmoi.toml`.

<br>

---

<br>

## New machine

<br>

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply Thadir
```

`chezmoi init` prompts for identity + email (work and personal), the two SSH
host aliases, and which `~/projects` subdirs belong to each account, and
writes them to `~/.config/chezmoi/chezmoi.toml` (never committed).

<br>

Then store any secrets — one-time, the value is prompted and never echoed:

<br>

```sh
# macOS
security add-generic-password -a "$USER" -s github_token -w

# Linux / WSL (whichever is installed)
pass insert cli/github_token
secret-tool store --label='github token' service github_token
```

<br>

Open a fresh shell. Until the token is stored, every shell prints one
yellow reminder line.

<br>

---

<br>

## Prompt

<br>

- **Left bar** (always shown): `host ⟩ user  directory` — seamless
  powerline blocks, `` end pointing at the cursor.

- **Right bar** (contextual): a pinned `` nose, then `cmd_duration`,
  language versions (node / go / rust / python), `aws`,
  `kubernetes` (k8s directories only), git state, exit code, and
  git branch + status + line metrics — each shown only when it has
  something to say.

<br>

---

<br>

## Editing

<br>

```sh
chezmoi edit ~/.zshrc          # edit the source, not the live file
chezmoi diff                   # preview
chezmoi apply                  # deploy

# after editing a live file directly:
chezmoi add ~/.zshrc && chezmoi git -- commit -am "…"
```

<br>

The prompt is the exception — edit `~/.config/starship-gen.py`, run it,
then `chezmoi add ~/.config/starship.toml`.
