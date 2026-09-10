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
in the repository. Do not enable the full `caveman` plugin just to get four helpers.
Use a verified four-skill-only mechanism if supported by the installed version;
otherwise skip the optional helpers and report why. A reply saying lite does not
prove the excluded skill body or plugin hooks cannot load later.

If the full plugin is already enabled, back up `config.toml`, remove `caveman`
from `plugins.enabled`, and report the change. Re-enable it only if the user
explicitly prefers the full plugin over this setup's exclusion list; report that
choice as a deviation, not a verified four-skill-only installation.

## Step 3: hooks and MCP

`~/.grok/config.toml`:

- `plugins.enabled`: remove `"context-mode"`.
- `[mcp_servers.codegraph]` and `[mcp_servers.context-mode]`: set `enabled = false`
  on each, or delete the tables. Keep CodeGraph only under the user-chosen
  existing-server exception; its automatic hook and routing rules still go.
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

Inspect reasoning defaults in the installed version and any launch wrapper.
Preserve model, effort, and window unless the user approves a change. For an
approved one-session trial, use a supported `--reasoning-effort` value; verify the
accepted levels with `grok --help` rather than assuming every model supports them.

## Verify

Native record: `~/.grok/sessions/<url-encoded-workdir>/<session-id>/chat_history.jsonl`
plus `prompt_history.jsonl`. After a fresh `grok -p` run with a fresh
`--session-id`:

- no `"rtk ` inside any executed command (the hook would have rewritten it);
- no unwanted `ctx_execute` or `codegraph_explore` calls, allowing only deliberate
  CodeGraph use under the recorded exception; pair this with live MCP inventory;
- native assistant model attribution where exposed, cross-checked with the JSON
  result's `modelUsage`. Earlier probes saw `grok-4.6-build` as a usage label for
  requested `grok-4.6`; do not infer authorship or dismiss a mismatch from a suffix
  alone. Record unexposed metadata as unverified.
