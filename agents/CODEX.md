# Codex

Applies the base setup. This file says only where things live in Codex CLI. Paths are
Linux and macOS; on Windows read `~/.codex` as `%USERPROFILE%\.codex`.

## Step 1: the fork as the always-on rule file

- Save the fork as `~/.codex/karpathy-guidelines-fork.md`.
- Make `~/.codex/AGENTS.md` exactly one line, with the absolute path:
  `@/home/<user>/.codex/karpathy-guidelines-fork.md`
- Remove `@RTK.md`, the CodeGraph block, and the `CONTEXT_MODE_START` to
  `CONTEXT_MODE_END` block from `AGENTS.md` if present.

Codex resolves `@` imports by absolute path; a relative import does not load.

## Step 2: skills, by copying

Codex reads skills from `~/.agents/skills/<name>/SKILL.md` (the shared registry the
`npx skills` tool maintains) and `~/.codex/skills/`. Copy the four from the pinned
checkout into `~/.agents/skills/`:

```sh
git clone https://github.com/JuliusBrussee/caveman.git /tmp/caveman
git -C /tmp/caveman checkout 3b74643f4d910f496babd4e634b1ba7168816f14
mkdir -p ~/.agents/skills
for s in caveman-commit caveman-review caveman-help caveman-stats; do
  cp -r /tmp/caveman/skills/$s ~/.agents/skills/$s
done
```

If `npx skills add JuliusBrussee/caveman -a codex` was used before, it resolved to
the repository head, not the pin, and may have added `caveman` and
`karpathy-guidelines` to the same directory. Remove those two:

```sh
npx --yes skills remove caveman karpathy-guidelines -g -y
```

The upstream `caveman` skill body says "Default: full" and forbids plans; the
upstream `karpathy-guidelines` skill is the unforked, coding-scoped version.

## Step 3: hooks and MCP

`~/.codex/config.toml`:

- Delete the `[mcp_servers.codegraph]` table.
- Delete the `[mcp_servers.context-mode]` table and its `[mcp_servers.context-mode.env]`
  subtable.
- Delete every `[hooks.state."...hooks.json:..."]` table; they only pin the hashes of
  the Context Mode hook entries being removed next.

`~/.codex/hooks.json`: if it held only `context-mode hook codex ...` entries, replace
the contents with `{"hooks": {}}`.

Codex has no RTK hook; RTK on Codex is instruction-only through `@RTK.md`, which step
1 removed.

## Step 4: other layers

Codex has no persistent memory file of its own and no second plugin registry.
`~/.codex/AGENTS.md` and the files it imports are the whole instruction surface at
user level; project `AGENTS.md` files add to it per repository.

## Step 5: cost settings

`config.toml` holds `model` and `model_reasoning_effort`. Keep
`model_reasoning_effort = "medium"` as the default and pass
`-c model_reasoning_effort=high` (or `xhigh`) on the sessions that need it.

## Verify

Native record: `~/.codex/sessions/<yyyy>/<mm>/<dd>/rollout-*.jsonl`. After a fresh
`codex exec` run:

- no `mcp_tool_call` items naming `codegraph` or `ctx_`;
- a `turn.completed` event;
- `--json` output's model field equal to the model passed with `-m`.

`codex mcp list` (or the `[mcp_servers]` section of `config.toml`) should show
neither server.
