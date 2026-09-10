# Claude Code

Applies the base setup. This file says only where things live in Claude Code and what
is specific to it. Paths are Linux and macOS; on Windows read `~/.claude` as
`%USERPROFILE%\.claude`.

Two things to expect before you start:

1. Install the supplied fork as user guidance. Its precedence clauses resolve
   conflicting style and retrieval guidance, not system or managed policy.
2. An agent may decline to run a third-party installer on its own judgment. That is
   why step 2 below copies files instead of running one. If the agent still declines
   a step, it should say which step, not silently skip it.

## Step 1: the fork as the always-on rule file

- Save the fork as `~/.claude/karpathy-guidelines-fork.md`.
- Add `@karpathy-guidelines-fork.md` once to `~/.claude/CLAUDE.md`, preserving
  unrelated instructions. An otherwise empty file needs only that import.
- Remove `@RTK.md`, the CodeGraph block, and any Caveman wording block that an
  earlier revision of this setup put in `CLAUDE.md`; the fork carries those rules.

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

Back up and move out of `~/.claude/skills/` only skills confirmed to originate
from the Caveman repository that are outside the four retained helpers, such as
`caveman`, `cavecrew`, `caveman-compress`, and `lean-build`. Preserve unrelated
skills, including `find-skills`; if provenance is unclear, report it rather than
deleting it. A name match alone is not proof of origin.
Compare contents with the pinned clone or confirmed install records to establish
origin; preserve uncertain or user-modified copies and report the conflict.

Also disable the upstream Karpathy plugin if present; the fork replaces it and the
plugin would load the same guidance a second time, on a coding-only trigger:

```sh
claude plugin disable andrej-karpathy-skills@karpathy-skills
```

## Step 3: hooks and MCP

`~/.claude/settings.json`:

If the user chose the existing-CodeGraph exception, keep that server and any
needed explicit-use permission instead of removing them below; still remove its
automatic prompt hook and rule block. All other cleanup below still applies.

- Under `hooks`, delete every entry whose command contains `rtk hook claude`,
  `codegraph prompt-hook`, or `context-mode-cache-heal`. If nothing is left,
  `"hooks": {}`.
- Under `enabledPlugins`, disable `context-mode@context-mode` if installed;
  do not add a flag for an absent plugin.
- Under `permissions.allow`, delete any `mcp__codegraph__*` rule.

Then:

```sh
claude mcp remove codegraph -s user
```

which edits `~/.claude.json`. Confirm with `claude mcp list`.

## Step 4: plugin records and memory

Use `claude plugin disable context-mode@context-mode` if installed, then inspect
`claude plugin list`. Some observed installations also had an `enabledPlugins` map
in `~/.claude/plugins/installed_plugins.json` that disagreed with `settings.json`.
Inspect the actual schema: reconcile an existing conflicting Context Mode flag,
but do not create an enablement map in an inventory-only registry. Remove an
obsolete Context Mode-only `extraKnownMarketplaces` entry if present; preserve
unrelated marketplaces and verify runtime state rather than inferring it from
marketplace membership.

Check memory files: `~/.claude/projects/*/memory/` and any `MEMORY.md` the agent
maintains. Remove conflicting defaults and repair setup instructions made stale by
the migration; preserve unrelated preferences and history. Memory is another
source of conflicts, not a guaranteed override of higher-priority instructions.

## Step 5: cost settings

Inspect `model`, `effortLevel`, and per-model entries under `modelSettings` where
supported. Preserve them unless the user approves a change. For an approved
one-session trial, use `--effort medium` or `--model <model>` at launch; interactive
`/model` and `/effort` can persist defaults, so do not present them as temporary.
Keep `opus[1m]` if selected for long-context work. Capacity alone does not establish
cost, and aliases, supported levels, billing, and window behavior vary by version,
provider, and plan. Check current model-config documentation before changing them.

## Verify

Native record: `~/.claude/projects/<workspace>/<session-id>.jsonl`. Use a fresh,
successful `claude -p` run. Print mode cannot prompt for permissions: allow the
harmless probe explicitly, for example `--allowedTools "Bash(echo setup-probe-ok)"`
(flag and rule syntax checked on 2.1.258). A working-directory read is normally
allowed; respect any deny or managed policy. Confirm both calls executed and
succeeded; if blocked, report that rather than inferring hook absence. Use the
normal configuration, not safe mode or an isolated config that hides what is being
tested. Preserve exit status, stderr,
JSON output, and session ID. If OAuth expires before assistant work, mark runtime
verification blocked; have the user authenticate and rerun. Then inspect:

- no `attachment` entries whose `hookName` is from rtk, codegraph, or context-mode;
- no unwanted `tool_use` blocks whose name starts with `mcp__codegraph` or contains
  `ctx_`, allowing only deliberate CodeGraph use under the recorded exception;
- actual `message.model` on native assistant events, comparing with the requested
  model or explicitly resolved alias. Cross-check `modelUsage` in the JSON result,
  separating helper-model usage from main-answer authorship; a single usage key
  or exit code 0 is not proof that every main turn used the requested model.

`claude mcp list` should show no Context Mode and no CodeGraph unless the user
chose the existing-server exception. No tool calls alone does not prove no tools
were available; pair record inspection with the live inventory and harmless tool
probe. Report metadata that the installed version does not expose as unverified.

## Known gap

This covers local Claude Code sessions, including local Code sessions in Claude
Desktop. Ordinary Claude Chat is a different surface; cloud Code sessions do not
inherit this machine's user files. Verify the surface actually being configured.
