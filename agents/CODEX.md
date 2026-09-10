# Codex

Applies the base setup. This file says only where things live in Codex CLI. Paths are
Linux and macOS; on Windows read `~/.codex` as `%USERPROFILE%\.codex`.

## Step 1: the fork as the always-on rule file

- Save the fork as `~/.codex/karpathy-guidelines-fork.md`.
- Embed the fork once in the active global instruction file, preserving unrelated
  content: `~/.codex/AGENTS.override.md` if present and nonempty, otherwise
  `~/.codex/AGENTS.md`. Respect `CODEX_HOME` if configured. Do not substitute an
  unverified `@path` import for the actual guidance.
- Replace an earlier revision's lone `@.../karpathy-guidelines-fork.md` import
  line with the embedded text and remove any earlier Caveman wording block. Also
  remove `@RTK.md`, the CodeGraph block, and the `CONTEXT_MODE_START` to
  `CONTEXT_MODE_END` block from the active global instruction file if present.

Confirm the full fork is present in the fresh session's instructions, not merely
a path the model might choose to read.

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

If an earlier installer was used, inspect both skill directories. Back up and move
out excluded skills of confirmed Caveman origin and the confirmed upstream
`karpathy-guidelines` skill replaced by the fork. Preserve unrelated skills and
report uncertain provenance. `~/.agents/skills` is shared: do not remove a copy
used by an agent outside the requested scope without approval.

The upstream `caveman` skill body says "Default: full" and forbids plans; the
upstream `karpathy-guidelines` skill is the unforked, coding-scoped version.

## Step 3: hooks and MCP

`~/.codex/config.toml`:

- Delete `[mcp_servers.codegraph]` unless the user chose the existing-server
  exception; in that case preserve the server, not automatic routing text/hooks.
- Delete the `[mcp_servers.context-mode]` table and its `[mcp_servers.context-mode.env]`
  subtable.
- Remove only `[hooks.state."...hooks.json:..."]` entries confirmed to refer to
  the Context Mode hooks being removed next; preserve unrelated hook state.

`~/.codex/hooks.json`: if it held only `context-mode hook codex ...` entries, replace
the contents with `{"hooks": {}}`. Otherwise remove only those entries.

Codex has no RTK hook; RTK on Codex is instruction-only through `@RTK.md`, which step
1 removed.

## Step 4: other layers

Inspect the first nonempty of `~/.codex/AGENTS.override.md` and
`~/.codex/AGENTS.md`, respecting `CODEX_HOME`, plus the effective settings and any
additional memory/instruction surfaces exposed by the installed version. Project
`AGENTS.md` files add guidance per repository. Do not assume an import was expanded.

## Step 5: cost settings

Inspect `model` and `model_reasoning_effort` in `config.toml`; preserve them unless
the user approves a change. For an approved one-session trial, use
`-c model_reasoning_effort=medium` only if that model supports the level. Do not
infer equal quality from a lower-effort run or silently change the model/window.

## Verify

Native record: `~/.codex/sessions/<yyyy>/<mm>/<dd>/rollout-*.jsonl`. After a fresh
`codex exec` run:

- no unwanted `mcp_tool_call` items naming `codegraph` or `ctx_`, allowing only
  deliberate CodeGraph use under the recorded exception;
- a `turn.completed` event;
- actual model attribution in the native record, compared with `-m` or its resolved
  alias. Event fields vary by version; do not invent a model field in `--json`
  output or treat requested session metadata as proof of every turn's author.

Check `codex mcp list`, not only config text: no Context Mode, and no CodeGraph
unless the user chose the existing-server exception. Apply the successful-session
and harmless-tool checks; missing runtime metadata remains unverified.
