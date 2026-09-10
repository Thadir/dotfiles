# dotfiles

Personal shell + terminal configuration, managed with
[chezmoi](https://chezmoi.io).

Runs on **macOS (Apple Silicon)**, **Linux**, and **Windows via WSL2**. The
macOS-only bits (the Homebrew bundle) guard themselves and skip elsewhere.

<br>

- **Set up a clean machine:** [`docs/setup.md`](docs/setup.md)
- **Update a machine that already has this:** [`docs/update.md`](docs/update.md)

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
| `~/.config/starship-gen.py` | `dot_config/starship-gen.py` | Generates `starship.toml`; injects Nerd Font glyphs via `chr()` because editors drop them. Run bare to write the live config, or pass a path / set `STARSHIP_TOML_OUT` to render elsewhere first. Then `starship print-config 2>&1 \| grep -iE 'warn\|error'`. |
| `~/.gitconfig` | `dot_gitconfig.tmpl` | **Personal identity is the default.** `includeIf` switches to the employer / EMU / customer account per directory (see [Git identities](#git-identities)). |
| `~/.config/git/{employer,emu}.inc` | `*.inc.tmpl` | Employer and EMU-account identity + an `insteadOf` rule routing `git@github.com:` through that account's SSH host alias. |
| `~/.config/git/customer-<slug>.inc` | _(generated)_ | One per `[[data.customer]]` in `chezmoi.toml`, written by `run_onchange_after_20-git-customer-includes.sh`. Stale ones are removed. |
| `~/.config/homebrew/Brewfile` | `dot_config/homebrew/Brewfile` | Every explicitly-installed formula, cask, VS Code extension and global npm package. Regenerate with `brew bundle dump --force --file=~/.config/homebrew/Brewfile`. |
| _(script)_ | `.chezmoiscripts/run_onchange_after_10-darwin-brew-bundle.sh.tmpl` | macOS only. Installs Homebrew if missing, then runs `brew bundle` — re-runs automatically whenever the Brewfile changes. |
| `~/Library/…/ghostty/config.ghostty` | `private_Library/…` | Ghostty terminal: `font-family = "JetBrainsMono Nerd Font Mono"` (must be the exact family name the terminal exposes), `grapheme-width-method = legacy`, Catppuccin auto light/dark. |

<br>

Identity/email values are **not** in this repo — `chezmoi init` prompts for
them (`.chezmoi.toml.tmpl`) and writes them to `~/.config/chezmoi/chezmoi.toml`.

<br>

---

<br>

## Git identities

<br>

**Personal is the default.** A work identity never applies unless the repo sits
under a directory you've mapped to it, so cloning something in a random
directory commits as you.

<br>

| Identity | Applies to | Source |
| --- | --- | --- |
| **personal** | everything not matched below | inline in `~/.gitconfig` |
| **employer** | `employer_dirs` subdirs of the projects root | `~/.config/git/employer.inc` |
| **EMU account** | `emu_dirs` subdirs | `~/.config/git/emu.inc` |
| **customer `<slug>`** | that customer's directory | `~/.config/git/customer-<slug>.inc` (generated) |

<br>

`~/.gitconfig` and the `.inc` files are all generated from
`~/.config/chezmoi/chezmoi.toml` (local only, never committed — it holds the
only copy of your names, emails and host aliases). Check which identity a repo
uses with `git -C <repo> config user.email`.

<br>

Adding a **customer account** is a small edit to `chezmoi.toml` plus
`chezmoi apply` — see [`docs/setup.md`](docs/setup.md#adding-a-customer-account).

<br>

---

<br>

## Prompt

<br>

- **Left bar** (always shown): `host ⟩ user  directory` — seamless
  powerline blocks, `` end pointing at the cursor. A padlock ()
  prefixes the host block when the session is privileged (SSH or root),
  driven by `$STARSHIP_PRIV` which `~/.zshrc` sets once at startup.

- **Right bar** (contextual): a pinned `` nose, then `cmd_duration`,
  language versions (node / go / rust / python), `aws`,
  `kubernetes` (k8s directories only), git state, git branch + status +
  line metrics, and — at the far right — a solid **red block** with the
  exit code when the last command failed. Each shown only when it has
  something to say.

- **Glyphs** are all Nerd Font (Powerline / Font Awesome) or plain ASCII —
  no emoji, no East-Asian-ambiguous-width symbols in the always-shown
  parts. That keeps zsh's width math and the terminal's in agreement, so
  the prompt doesn't stack or smear on redraw / window resize (the usual
  culprit is an emoji that's 1 cell to zsh but 2 in the terminal).

<br>

---

<br>

## Editing

<br>

```sh
chezmoi edit ~/.zshrc          # edit the source, not the live file
chezmoi diff                   # preview
chezmoi apply                  # deploy
chezmoi git -- add -A && chezmoi git -- commit -m "…" && chezmoi git -- push
```

<br>

The prompt is the exception — it's generated. Edit
`~/.config/starship-gen.py`, run it (optionally to a temp path first to
eyeball), then `chezmoi add ~/.config/starship.toml ~/.config/starship-gen.py`.

<br>

More detail — previewing, rolling back, migrating — is in
[`docs/update.md`](docs/update.md).
