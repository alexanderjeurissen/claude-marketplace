# alexanderjeurissen/claude-marketplace

A personal Claude Code marketplace. Three plugins, split by where they are useful rather than by
subject:

| Plugin | What it is | Where it helps |
| --- | --- | --- |
| `workspace` | The per-issue worktree engine (`workspace`, `cmux-spawn-work`), the four `/workspace` commands, the read-only-traversal approval hook, and the workflow guide as a skill | Local only — it operates on a submodule hub, so it has nothing to do in a remote session |
| `cmux` | The `cmux-*` skill family | Local only |
| `house-style` | Conventions that should hold everywhere, starting with the Mermaid diagram style | Local **and** remote — this is the half a repo enables so a web session knows the house rules |

## Install

```sh
claude plugin marketplace add alexanderjeurissen/claude-marketplace
claude plugin install workspace@alexanderjeurissen
claude plugin install cmux@alexanderjeurissen
claude plugin install house-style@alexanderjeurissen
```

Then restart Claude Code. `claude plugin update <name>@alexanderjeurissen` picks up later changes.

For a repo that should always carry the house style, commit it instead of installing by hand:

```json
{
  "extraKnownMarketplaces": [
    { "name": "alexanderjeurissen", "type": "github", "owner": "alexanderjeurissen", "repo": "claude-marketplace" }
  ],
  "enabledPlugins": { "house-style@alexanderjeurissen": true }
}
```

> **Unverified in remote sessions.** Repo-committed `enabledPlugins` did not install anything in a
> Claude Code on the web session during testing, while the same marketplace was reachable over the
> network. If that reproduces, a repo needs a `SessionStart` hook running
> `claude plugin marketplace add …` and `claude plugin install -s local -y house-style@alexanderjeurissen`
> instead. Settle this in one repo before rolling it out.

## Why a marketplace and not dotfiles

These components used to live in `alexanderjeurissen/dotfiles` and reach machines through `rcup`,
which links `~/.claude/`. That still works and is not going away tomorrow. A marketplace adds three
things `rcup` cannot: a version to pin and update, per-repo opt-in instead of everything being
global, and — for `house-style` — reach into remote sessions, where dotfiles never runs.

## Layout

```
.claude-plugin/marketplace.json
plugins/workspace/     bin/ commands/ hooks/ skills/workspace-workflow/
plugins/cmux/          skills/cmux*, 8 skills
plugins/house-style/   skills/mermaid-style/
```

`bin/` is added to the Bash tool's PATH while the plugin is enabled, which is what replaces
`~/.scripts`. (That mechanism is unavailable for plugins distributed through claude.ai organization
settings — a git marketplace is required, which is what this is.)
