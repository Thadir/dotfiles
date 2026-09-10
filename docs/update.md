# Update — an existing machine

Pulling new changes onto a machine that already runs these dotfiles.
For a machine that has never had them, see [`setup.md`](setup.md).

<br>

---

<br>

## The normal case

<br>

```sh
chezmoi update        # git pull in ~/.local/share/chezmoi, then chezmoi apply
```

Preview first if you want:

```sh
chezmoi update --dry-run --verbose
# or, after a manual pull:
chezmoi git -- pull && chezmoi diff
```

<br>

The prompt config (`~/.config/starship.toml`) and `~/.gitconfig` regenerate
automatically. `~/.config/chezmoi/chezmoi.toml` — your local identity data — is
never touched by an update.

<br>

---

<br>

## When the prompts changed

<br>

chezmoi prints a warning when `.chezmoi.toml.tmpl` has changed:

```
config file template has changed, run chezmoi init to regenerate config file
```

Then:

```sh
chezmoi init          # re-runs the prompts; existing answers pre-fill, you
                      # only answer anything genuinely new
chezmoi apply
```

`chezmoi init` never loses data you already have — including any
`[[data.customer]]` blocks you added by hand.

<br>

---

<br>

## Migrating from the old single-work-account config

<br>

Older machines stored one `work` identity (`name`, `email_work`,
`ssh_host_work`, `work_dirs`). The current model splits work into **employer**
and **EMU** and makes **personal** the default.

<br>

```sh
chezmoi update        # get the new templates
chezmoi init          # the employer fields pre-fill from the old work values;
                      # answer the new EMU prompts (or leave the EMU email blank)
chezmoi apply
```

<br>

`chezmoi apply` then:

- rewrites `~/.gitconfig` so the default identity is **personal**;
- replaces `~/.config/git/{personal,work}.inc` with `employer.inc` / `emu.inc`;
- keeps every existing repo committing exactly as before, as long as its
  directory is still listed in `employer_dirs`.

<br>

A backup of the pre-migration config is worth keeping:

```sh
cp ~/.config/chezmoi/chezmoi.toml ~/.config/chezmoi/chezmoi.toml.bak
```

<br>

---

<br>

## Changing something yourself

<br>

```sh
chezmoi edit ~/.zshrc          # edit the SOURCE, not the live file
chezmoi diff                   # preview
chezmoi apply                  # deploy locally
chezmoi git -- add -A
chezmoi git -- commit -m "..."
chezmoi git -- push            # opens a PR flow; merging auto-tags a release
```

<br>

The prompt is the exception — it's generated:

```sh
chezmoi edit ~/.config/starship-gen.py
python3 ~/.config/starship-gen.py /tmp/preview.toml   # eyeball it first
python3 ~/.config/starship-gen.py                     # write the live config
chezmoi add ~/.config/starship.toml ~/.config/starship-gen.py
```

<br>

---

<br>

## Rolling back

<br>

```sh
chezmoi git -- log --oneline
chezmoi git -- checkout <good-commit> -- .
chezmoi apply
```

Or check out an earlier release tag in `~/.local/share/chezmoi` and
`chezmoi apply`.
