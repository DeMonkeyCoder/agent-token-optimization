# Cursor (CLI and desktop IDE)

Applies the base setup. If Claude Code is installed on the same machine, its hooks
run inside Cursor (see step 3). Removing them means editing Claude Code's files:
if Claude Code was not in the requested scope, ask first. Without approval, report
Cursor's hook state as blocked rather than editing another agent's configuration.
The CLI (`cursor-agent`) and the desktop IDE share the same rule, MCP, and hook
files, so one pass covers both.

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

In the desktop IDE, Customize > Rules > User Rules apply to every project. Inspect
onboarding profile text and remove only setup-related conflicts; preserve unrelated
user preferences. Choose one carrier: project rules, which the IDE reads too, or,
for IDE-only use, the full fork in User Rules. Do not use both, or the fork loads
twice in the IDE. The Output style section alone does not install the guidelines.
Verify loading on the surface you chose; do not assume IDE User Rules reach the CLI.

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
  `context-mode` from `mcpServers`, keeping CodeGraph only under the user-chosen
  existing-server exception, without automatic routing guidance/hooks. On a fresh
  install neither file exists; creating `~/.cursor/mcp.json` as `{"mcpServers": {}}`
  is harmless.
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
`~/.claude/settings.json` first, within the consent rule above.

## Step 4: other layers

No persistent memory file at user level. Check User Rules (IDE) as in step 1.

## Step 5: cost settings

The tested free plan allowed only `--model auto`; current availability can differ.
Preserve the selected model, effort, and window unless the user approves a change.
Use the installed CLI's model list or IDE picker for an approved trial; do not
guess model suffixes or assume an Auto selection proves which model answered.

## Verify

CLI: use a fresh `cursor-agent -p` session with the current model selection and
`--output-format json`. A text-only `--mode ask` smoke test cannot verify shell
hooks; exercise harmless shell/read work in the normal supported tool mode too.
The result carries `session_id`. Native records:

- transcript: `~/.cursor/projects/<workspace>/agent-transcripts/<session-id>/`
- chat store: `~/.config/cursor/chats/<workspace-hash>/<session-id>/store.db`, where
  `providerOptions.cursor.modelName` names the model that actually answered. Read
  it from there; the `--model` flag is not proof on a free plan.

IDE: the same transcript directory is written for IDE chats. Customize > Hooks
shows configured and fired Cursor hooks, but not Claude Code hooks it loaded; the
transcript and a marker file are the only evidence for those.
