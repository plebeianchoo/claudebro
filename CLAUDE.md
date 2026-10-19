# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A master repo for Jeremy's Claude Code setup. It holds no code of its own: it pins
three separate repos as git submodules, so one `git clone --recurse-submodules`
brings everything. All real work happens inside a submodule.

| Submodule | Contents | Details |
|---|---|---|
| `claudebro_panes/` | tmux two-pane layout (Claude top, shell bottom), Nord theme, popups, "Claude needs you" bell hooks | `claudebro_panes/README.md` ("How it works, and why" explains the non-obvious design choices) |
| `claude-statusline/` | single-script statusline (`statusline.sh`, bash + jq) | `claude-statusline/README.md` |
| `claude-mods/` (**private**) | local plugin marketplace `claudebro-mods`: `cat-clawd-duo` and `spinner` (own forks of upstream spinner mods), `peek` (own, no upstream), `templates/blink-spinner` (starter) | `claude-mods/README.md` |

Submodule URLs are relative (`../<repo>.git`), so they resolve against however
this repo was cloned. The clone must live at `~/code/claudebro`: `~/.tmux.conf`, the
shell rc and `~/.claude/settings.json` reference files in it by absolute path,
so don't move or rename files that are sourced from outside without updating
the installers.

## Committing: two steps, always

Each submodule is its own repo. Commit and push **inside** the submodule first,
then record the new pointer here:

```sh
cd ~/code/claudebro/<sub> && git commit -am "…" && git push
cd ~/code/claudebro && git add <sub> && git commit -m "Bump <sub>" && git push
```

Before committing in a submodule, check it is on `main` and not a detached HEAD
(`git submodule update` leaves them detached). Never bump the master to a
submodule commit that hasn't been pushed. New submodules: `git submodule add
../<repo>.git <repo>`, then add a row to the table in `README.md`.

## Per-submodule workflows

**claudebro_panes** — no build or tests. `./install.sh` appends marked blocks to
`~/.tmux.conf` and the shell rc, and optionally copies `claude/USAGE.md` into
`~/.claude/CLAUDE.md` and installs the bell hooks into `~/.claude/settings.json`
(via `claude/hooks.sh`). `USAGE.md` is *copied*, not linked: after editing it,
re-run `./install.sh` and answer `y` to refresh. Panes are addressed by pane id
(`%N`) and border labels key off a per-pane `@role` option — keep it that way
(index addressing and pane titles both break; see that README).

**claude-statusline** — test a change by piping a sample payload:

```sh
echo '{"workspace":{"current_dir":"/tmp/demo"},"model":{"display_name":"Opus 5"},
"cost":{"total_cost_usd":1.23},"context_window":{"remaining_percentage":62},
"rate_limits":{"five_hour":{"used_percentage":38,"resets_at":9999999999}}}' | ./statusline.sh
```

Check all three rendering tiers: `COLORTERM=truecolor`, unset (256-colour), and
`CLAUDE_STATUSLINE_ASCII=1`.

**claude-mods** — `cat-clawd-duo/` and `spinner/` are Jeremy's own forks
(of cat-spinner + clawd-spinner, and hoobnn's spinner) and are edited in place.
`peek/` is Jeremy's own mod with no upstream; the same validate/test/reinstall steps apply to it.
Test a change, then reinstall:

```sh
claude plugin validate <mod>/.claude-plugin/plugin.json && claude plugin test <mod>
claude plugin marketplace update claudebro-mods && claude plugin update <mod>@claudebro-mods
```

The originals live as reference clones in `upstream/<name>/` (gitignored).
`upstream/check.sh` triages each upstream commit since the baseline in
`upstream/sources.tsv` (docs / release / ported / unused / clean / conflict), using
`upstream/maps/<name>.sed` to map upstream paths onto the mod. A monthly timer runs it
with `--notify` → Telegram. Port with the `port-upstream` skill (`--port`, security
review, validate + test, commit, `--mark`); details in `claude-mods/upstream/README.md`.
