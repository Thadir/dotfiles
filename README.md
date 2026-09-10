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
| `~/.config/starship-gen.py` | `dot_config/starship-gen.py` | Generates `starship.toml`; injects Nerd Font glyphs via `chr()` because editors drop them. Run bare to write the live config, or pass a path / set `STARSHIP_TOML_OUT` to render elsewhere first. Then `starship print-config 2>&1 \| grep -iE 'warn\|error'`. |
| `~/.gitconfig` | `dot_gitconfig.tmpl` | **Personal identity is the default.** `includeIf` switches to the employer / EMU / customer account per directory (see [Git identities](#git-identities)). |
| `~/.config/git/{employer,emu}.inc` | `*.inc.tmpl` | Employer and Capgemini-EMU identity + an `insteadOf` rule routing `git@github.com:` through that account's SSH host alias. |
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

## New machine

<br>

Prerequisites (Linux / WSL): `git`, `curl`, `zsh`, and `~/.ssh/config` with the
host aliases and keys you'll enter at the prompts (`github.com-personal`,
`github.com-sogeti`, `github.com-cgemu`, …). macOS has git/curl/zsh already.

<br>

```sh
sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply Thadir
```

`chezmoi init` prompts for the **personal**, **employer** and **Capgemini EMU**
identities (name / email / SSH host alias), the projects root, and which
`~/projects` subdirs use the employer and EMU accounts. Answers are written to
`~/.config/chezmoi/chezmoi.toml` (never committed). Customer accounts are added
afterwards — see [Git identities](#git-identities).

<br>

On a machine already set up with the **old** single-`work` config, just re-run
`chezmoi init` (no `--apply`): the new prompts pre-fill from the old values, and
you only answer the genuinely new ones (the EMU email, mostly). Then
`chezmoi apply`.

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

## Git identities

<br>

**Personal is the default.** A work identity never applies unless the repo sits
under a directory you've mapped to it, so cloning something in a random
directory commits as you, not as a customer.

<br>

| Identity | Applies to | Source |
| --- | --- | --- |
| **personal** | everything not matched below | inline in `~/.gitconfig` |
| **employer** (Sogeti / Capgemini corporate) | `employer_dirs` subdirs of the projects root | `~/.config/git/employer.inc` |
| **Capgemini EMU** (customer work via Capgemini) | `emu_dirs` subdirs | `~/.config/git/emu.inc` |
| **customer `<slug>`** | that customer's directory | `~/.config/git/customer-<slug>.inc` (generated) |

<br>

`~/.gitconfig` and the `.inc` files are all generated from
`~/.config/chezmoi/chezmoi.toml`. Check which identity a repo uses with
`git -C <repo> config user.email`.

<br>

### Adding a customer account

<br>

Edit `~/.config/chezmoi/chezmoi.toml` and add a block per customer, then apply:

<br>

```toml
[[data.customer]]
    slug     = "acme"                 # also the directory name, and the .inc suffix
    email    = "m.cremer@acme-ext.com"
    # optional:
    name     = "Martijn Cremer"       # defaults to the personal name
    dir      = "~/work/acme"          # defaults to <projects_root>/<slug>
    ssh_host = "github.com-acme"       # adds an insteadOf route; omit if not needed
```

<br>

```sh
chezmoi apply
```

<br>

That regenerates `~/.gitconfig` (adds the `includeIf`) and writes
`~/.config/git/customer-acme.inc`. Removing the block and re-applying deletes the
`.inc` again. Edits to `chezmoi.toml` survive `chezmoi init`.

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

# after editing a live file directly:
chezmoi add ~/.zshrc && chezmoi git -- commit -am "…"
```

<br>

The prompt is the exception — edit `~/.config/starship-gen.py`, run it,
then `chezmoi add ~/.config/starship.toml`.
