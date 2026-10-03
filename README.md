# claudebro

My custom Claude Code setup, in one place. Each piece lives in its own repo;
this one ties them together as git submodules, so a single clone brings
everything.

| Repo | What it is |
|---|---|
| [`claudebro_panes`](https://github.com/plebeianchoo/claudebro_panes) | Two-pane tmux layout for Claude Code (Claude on top, a shell below), plus Nord theme, popups (lazygit, btop, markdown, tldr, session picker) and "Claude needs you" alerts |
| [`claude-statusline`](https://github.com/plebeianchoo/claude-statusline) | Custom Claude Code statusline: cwd, branch, model, cost, context and a 5h usage bar |
| `claude-mods` (private) | My own Claude Code mods, as a local plugin marketplace. Private, so a clone without access skips it |

## Set up on a new machine

```sh
git clone --recurse-submodules git@github.com:plebeianchoo/claudebro.git ~/claudebro

# tmux layout, popups, theme, alerts (asks before touching CLAUDE.md / settings.json)
~/claudebro/claudebro_panes/install.sh

# statusline
mkdir -p ~/.claude
ln -sf ~/claudebro/claude-statusline/statusline.sh ~/.claude/statusline.sh
# then set "statusLine": {"type": "command", "command": "~/.claude/statusline.sh"}
# in ~/.claude/settings.json and restart Claude Code
```

Keep the clone at `~/claudebro`: tmux, the shell rc and Claude's settings
point into it by absolute path. Each sub-repo's README has the details.

The submodule URLs are relative (`../claudebro_panes.git`), so they follow
however this repo was cloned: SSH here, HTTPS for anyone without a key.

## Working on a sub-repo

Each folder is a full repo of its own. Commit and push inside it as usual,
then record the new version here:

```sh
cd ~/claudebro/claudebro_panes
git commit -am "…" && git push

cd ~/claudebro
git add claudebro_panes && git commit -m "Bump claudebro_panes" && git push
```

## Updating another machine

```sh
cd ~/claudebro
git pull --recurse-submodules
git submodule update --init --recursive
```

To pull each sub-repo's latest `main` even if this repo hasn't been bumped
yet: `git submodule update --remote --merge`.

## Adding another repo

```sh
cd ~/claudebro
git submodule add ../<repo>.git <repo>
git commit -m "Add <repo>" && git push
```

Then add a row to the table above.
