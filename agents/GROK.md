# Grok CLI

Applies the base setup. This file says only where things live in Grok CLI.

## Step 1: the fork as an always-on rule

Every `*.md` file in `~/.grok/rules/` is loaded into every session. Save the fork
as `~/.grok/rules/karpathy-guidelines-fork.md`. No import line is needed.

Delete `~/.grok/rules/rtk.md`, `~/.grok/rules/codegraph.md`, and
`~/.grok/rules/context-mode.md` if present. If a `~/.grok/rules/caveman.md` exists
from an earlier setup, either delete it (the fork carries the rules) or make sure it
says lite and does not forbid plans; two files saying slightly different things is
how drift starts.

## Step 2: skills

Grok installs skills through plugins from `~/.grok/config.toml`
`[[marketplace.sources]]` entries, and the caveman marketplace exposes every skill
in the repository. Prefer not to enable the `caveman` plugin. If it is enabled and
you want the four helpers, keep the plugin but confirm the `caveman` skill body is
not loaded in a fresh session (ask for the caveman level; if the reply says full,
the body is loading).

## Step 3: hooks and MCP

`~/.grok/config.toml`:

- `plugins.enabled`: remove `"context-mode"`.
- `[mcp_servers.codegraph]` and `[mcp_servers.context-mode]`: set `enabled = false`
  on each, or delete the tables. Disabling keeps re-enabling to one flag flip.
- Delete the `[[hooks.PreToolUse]]` block whose command runs an RTK adapter
  (`rtk-hook.py` or similar), together with its `[[hooks.PreToolUse.hooks]]` entry.
- `~/.grok/hooks/rtk-hook.py` and `~/.grok/hooks/rtk.json` can stay on disk; with the
  block gone they are inert.

The `[[marketplace.sources]]` entry for `context-mode` can stay; a source is not a
plugin.

## Step 4: other layers

Grok has no persistent memory file at user level. The rules directory and enabled
plugins are the whole instruction surface.

## Step 5: cost settings

`--reasoning-effort low|medium|high|xhigh` per session. There is no persistent
default in `config.toml`; whatever wrapper or alias you use to launch Grok should not
hard-code `xhigh`.

## Verify

Native record: `~/.grok/sessions/<url-encoded-workdir>/<session-id>/chat_history.jsonl`
plus `prompt_history.jsonl`. After a fresh `grok -p` run with a fresh
`--session-id`:

- no `"rtk ` inside any executed command (the hook would have rewritten it);
- no `ctx_execute` or `codegraph_explore` tool names;
- the JSON result's `modelUsage` map has one key. Grok labels its usage bucket with a
  `-build` suffix (`grok-4.6-build` for `--model grok-4.6`); that is the same model,
  not a substitution.
