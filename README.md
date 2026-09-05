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

For a repo that should always carry the house style, commit a `SessionStart` hook — **not**
`enabledPlugins`:

```json
{
  "hooks": {
    "SessionStart": [
      { "matcher": "startup|resume|clear|compact",
        "hooks": [ { "type": "command", "command": "bash \"$CLAUDE_PROJECT_DIR/.claude/hooks/install-house-style.sh\"", "timeout": 30 } ] }
    ]
  }
}
```

```sh
#!/usr/bin/env bash
set -u
command -v claude >/dev/null 2>&1 || exit 0
claude plugin list 2>/dev/null | grep -q 'house-style@alexanderjeurissen' && exit 0
claude plugin marketplace add alexanderjeurissen/claude-marketplace >/dev/null 2>&1
claude plugin install house-style@alexanderjeurissen -s local -y >/dev/null 2>&1
exit 0
```

Measured at 3.0s cold and 0.4s warm, idempotent, always exit 0.

### Two things this cannot do

**Repo-committed `extraKnownMarketplaces` + `enabledPlugins` installs nothing.** Tested in both the
documented array form and the older object form, by running a real session in a project declaring
them: the marketplace registry stayed empty while the same settings file's other keys were honoured.
That is why the hook above exists.

**A plugin is not available to the turn that installs it.** A session whose `SessionStart` hook
installs the plugin answers "no" when asked for the skill; the next one answers "yes". It does
arrive on a later turn of the same session. So the hook is fine for an ordinary multi-turn session
and useless for a one-shot automated run — and anything that must be reliable on turn one belongs in
the repo's own `.claude/skills/` instead, committed.

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
