# Claude Code

Applies the base setup. This file says only where things live in Claude Code and what
is specific to it. Paths are Linux and macOS; on Windows read `~/.claude` as
`%USERPROFILE%\.claude`.

Two things to expect before you start:

1. The agent will notice that the fork claims precedence over other instructions.
   That is deliberate; the user asked for it. Apply it as written.
2. An agent may decline to run a third-party installer on its own judgment. That is
   why step 2 below copies files instead of running one. If the agent still declines
   a step, it should say which step, not silently skip it.

## Step 1: the fork as the always-on rule file

- Save the fork as `~/.claude/karpathy-guidelines-fork.md`.
- Make `~/.claude/CLAUDE.md` exactly one line: `@karpathy-guidelines-fork.md`
- Remove `@RTK.md` and the CodeGraph block from `CLAUDE.md` if present.

Import syntax: `@path` on its own line, relative to the `CLAUDE.md` that contains it.
Do not put the caveman rules in `CLAUDE.md` a second time; the fork carries them.

## Step 2: skills, by copying

Claude Code loads skills from `~/.claude/skills/<name>/SKILL.md`. Copy the four
directories from the pinned checkout:

```sh
git clone https://github.com/JuliusBrussee/caveman.git /tmp/caveman
git -C /tmp/caveman checkout 3b74643f4d910f496babd4e634b1ba7168816f14
mkdir -p ~/.claude/skills
for s in caveman-commit caveman-review caveman-help caveman-stats; do
  cp -r /tmp/caveman/skills/$s ~/.claude/skills/$s
done
```

Do not install the caveman marketplace plugin (`claude plugin install
caveman@caveman`). It ships every skill in the repository plus a SessionStart hook
that injects the full-mode ruleset and a UserPromptSubmit hook, and its marketplace
config auto-discovers new skills on update. If it is already installed, disable it:

```sh
claude plugin disable caveman@caveman
```

and remove any of these from `~/.claude/skills/` if an earlier install put them
there: `caveman`, `cavecrew`, `caveman-compress`, `lean-build`, and anything else
outside the four.

Also disable the upstream Karpathy plugin if present; the fork replaces it and the
plugin would load the same guidance a second time, on a coding-only trigger:

```sh
claude plugin disable andrej-karpathy-skills@karpathy-skills
```

## Step 3: hooks and MCP

`~/.claude/settings.json`:

- Under `hooks`, delete every entry whose command contains `rtk hook claude`,
  `codegraph prompt-hook`, or `context-mode-cache-heal`. If nothing is left,
  `"hooks": {}`.
- Under `enabledPlugins`, set `"context-mode@context-mode": false`.
- Under `permissions.allow`, delete any `mcp__codegraph__*` rule.

Then:

```sh
claude mcp remove codegraph -s user
```

which edits `~/.claude.json`. Confirm with `claude mcp list`.

## Step 4: the second plugin registry

`~/.claude/plugins/installed_plugins.json` has its own `enabledPlugins` map and can
disagree with `settings.json`. Set `"context-mode@context-mode": false` there too.
Also remove `context-mode` from `extraKnownMarketplaces` in `settings.json` so an
update cannot bring it back.

Check memory files: `~/.claude/projects/*/memory/` and any `MEMORY.md` the agent
maintains. A line saying caveman is full overrides the fork.

## Step 5: cost settings

`settings.json` may hold `model` and `modelSettings` with per-model `effortLevel`.
Do not leave every model at the top effort. Set the working model's effort to a
middle value and raise it per session when the task warrants (`--effort` on the CLI,
or `/model` in the session). A 1M-context model selection (`opus[1m]` style) costs
more per turn; use it when the task needs the window.

## Verify

Native record: `~/.claude/projects/<workspace>/<session-id>.jsonl`. After a fresh
`claude -p` run:

- no `attachment` entries whose `hookName` is from rtk, codegraph, or context-mode;
- no `tool_use` blocks whose name starts with `mcp__codegraph` or contains `ctx_`;
- the `modelUsage` map in the `--output-format json` result has exactly one key, and
  it is the model you requested. The CLI exits 0 even when the provider substituted
  a different model mid-run, so read this field, do not trust the exit code.

`claude mcp list` should show neither `codegraph` nor `context-mode`.

## Known gap

The Claude desktop chat app and claude.ai do not read `~/.claude`. This setup covers
Claude Code sessions only.
