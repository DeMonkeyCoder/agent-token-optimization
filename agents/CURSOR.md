# Cursor (CLI and desktop IDE)

Applies the base setup. If Claude Code is installed on the same machine, clean
it up before Cursor: Cursor runs Claude Code's hooks (see step 3). The
CLI (`cursor-agent`) and the desktop IDE share the same rule, MCP, and hook files,
so one pass covers both.

## Step 1: the fork as an always-apply rule

Cursor has no user-level rules directory on disk. The always-on carrier is a
project rule, one per repository you work in:

- Save the fork as `<repo>/.cursor/rules/karpathy-guidelines-fork.mdc` with this
  exact header above the fork text:

```
---
alwaysApply: true
---
```

The extension must be `.mdc` and the header must be present. A `.md` file in
`.cursor/rules/` is silently ignored; this was tested, the agent answered NO when
asked whether it had the fork's precedence sentence. Do not rely on a
`description:` field for matching; that is the coding-scoped-skill problem again.

A project `AGENTS.md` at the repository root is the plain-markdown alternative and
is always read.

In the desktop IDE, Customize > Rules > User Rules apply to every project. The IDE's
onboarding writes profile text there ("prefers coding workflows" and similar); read
it and delete anything that sets a style or narrows scope. Pasting the fork's
Output style section into User Rules covers repositories without a project rule.

## Step 2: skills

Cursor reads skills from `<repo>/.cursor/skills/` and `~/.cursor/skills/`. Copy the
four from the pinned checkout into `~/.cursor/skills/`:

```sh
git clone https://github.com/JuliusBrussee/caveman.git /tmp/caveman
git -C /tmp/caveman checkout 3b74643f4d910f496babd4e634b1ba7168816f14
mkdir -p ~/.cursor/skills
for s in caveman-commit caveman-review caveman-help caveman-stats; do
  cp -r /tmp/caveman/skills/$s ~/.cursor/skills/$s
done
```

## Step 3: hooks and MCP

- `~/.cursor/mcp.json` and `<repo>/.cursor/mcp.json`: remove `codegraph` and
  `context-mode` from `mcpServers`, or leave CodeGraph's entry for the
  large-repository case in the base setup. On a fresh install neither file
  exists; creating `~/.cursor/mcp.json` as `{"mcpServers": {}}` is harmless.
- `~/.cursor/hooks.json` and `<repo>/.cursor/hooks.json`: remove any
  `beforeShellExecution` or `preToolUse` entry that runs `rtk`, and any
  `context-mode hook` entry. `{"hooks": {}}` if nothing is left.
- Context Mode's Cursor install is a local plugin symlinked at
  `~/.cursor/plugins/local/context-mode`. Remove the symlink and check
  Settings > Plugins in the IDE. Its README warns that a `hooks.json` install plus
  the plugin double-fires every hook, so check both places.

**Cursor runs Claude Code's hooks by default.** Tested on the CLI (2026.09.02) and
the IDE (3.19.13) with no Cursor setting touched: a `PreToolUse` hook present only in
`~/.claude/settings.json` fired on a Cursor shell call, and the IDE's Hooks tab
reported "no hooks configured" while it did. The documented opt-in ("Include
third-party Plugins, Skills, and other configs") was not a precondition on these
builds. An RTK or Context Mode hook left in the Claude file is live in Cursor even
when Cursor's own `hooks.json` is empty and its UI shows nothing. Clean
`~/.claude/settings.json` first.

## Step 4: other layers

No persistent memory file at user level. Check User Rules (IDE) as in step 1.

## Step 5: cost settings

Free plans allow `--model auto` only; named models are rejected. On paid plans, pick
the model per session with `--model` (CLI) or the composer's model picker (IDE) and
prefer a mid-effort variant for mechanical work. The `-high` and `-xhigh` suffixes
on model names are the effort setting.

## Verify

CLI: `cursor-agent -p --trust --mode ask --model auto --output-format json '<smoke
test question>'`. The result carries `session_id`. Native records:

- transcript: `~/.cursor/projects/<workspace>/agent-transcripts/<session-id>/`
- chat store: `~/.config/cursor/chats/<workspace-hash>/<session-id>/store.db`, where
  `providerOptions.cursor.modelName` names the model that actually answered. Read
  it from there; the `--model` flag is not proof on a free plan.

IDE: the same transcript directory is written for IDE chats. Customize > Hooks
shows configured and fired Cursor hooks, but not Claude Code hooks it loaded; the
transcript and a marker file are the only evidence for those.
