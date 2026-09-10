# Hermes

Applies the base setup. This file says only where things live in Hermes and what is
specific to it. Everything below is per profile: the default profile is
`~/.hermes`, named profiles are `~/.hermes/profiles/<name>`, and each has its own
`config.yaml`, prompt, plugins, skills, and memories. Repeat only for the profiles
the user requested, and use `hermes --profile <name>` for the commands.

## Step 1: the fork as a system-prompt section

Hermes has no rule-file import. The always-on carrier is a plugin that registers a
system-prompt section, which is rendered once per session and frozen into the cached
prefix. A skill is the wrong carrier: it loads only when its description matches
the task.

Create `<profile>/plugins/karpathy-guidelines/plugin.yaml`:

```yaml
name: karpathy-guidelines
version: "1.0.0"
description: Always-on behavioral guidelines, registered as a system prompt section.
author: local
manifest_version: 2
license: MIT
```

and `<profile>/plugins/karpathy-guidelines/__init__.py`:

```python
_GUIDELINES = """<paste the fork here, from "# Coding Guidelines" to the end>"""


def register(ctx):
    ctx.register_system_prompt_section(
        "karpathy-guidelines.principles",
        _GUIDELINES,
        position="after_memory",
        max_chars=4000,
    )
```

Paste the fork text verbatim inside the triple quotes. The tested API accepts a
maximum of 4000 characters per section; the supplied fork is 3789 ASCII characters.
This was checked on Hermes v0.20.4, local source `ad1ee7a3`. If the text grows,
check its length and the installed API rather than raising the cap blindly.
Overlong content is skipped entirely, not truncated; a successful file write can
therefore leave no loaded guidance. Then:

```sh
hermes --profile <name> plugins enable karpathy-guidelines
hermes --profile <name> plugins doctor karpathy-guidelines
```

The fork's "Output style" section carries the caveman lite rules, so the profile's
`agent.system_prompt` does not need a separate caveman block. If the prompt already
has one from an earlier setup, either delete it or make sure it says lite and does
not forbid plans; two versions drift.

Read the prompt with `hermes --profile <name> config get agent.system_prompt` and
set it with `config set`. Verify by `config get` again, not by reading `config.yaml`,
because layered config can differ from the file.

## Step 2: skills

Hermes skills live in `<profile>/skills/<category>/<name>/SKILL.md`. Copy the four
from the pinned checkout into `<profile>/skills/productivity/`:

```sh
git clone https://github.com/JuliusBrussee/caveman.git /tmp/caveman
git -C /tmp/caveman checkout 3b74643f4d910f496babd4e634b1ba7168816f14
mkdir -p <profile>/skills/productivity
for s in caveman-commit caveman-review caveman-help caveman-stats; do
  cp -r /tmp/caveman/skills/$s <profile>/skills/productivity/$s
done
```

The upstream installer (`npx -y github:JuliusBrussee/caveman -- --only hermes`)
installs seven skills including the `caveman` body. If it was used, move
`<profile>/skills/productivity/caveman/` out of the skill tree.

## Step 3: hooks, MCP, and plugins

Skip server removal only when the user chose the existing-CodeGraph exception;
still disable automatic prompt/command hooks below.

```sh
hermes --profile <name> mcp remove codegraph
hermes --profile <name> plugins disable rtk-rewrite
hermes --profile <name> plugins disable codegraph-context
```

The second and third report "not installed" on profiles that never had them; that is
fine. `hermes plugins list --plain --no-bundled` shows what is enabled.

Remove the paragraph beginning "Prefix shell/terminal commands with `rtk`" from
`agent.system_prompt` if present. Context Mode is not installable on Hermes; its
routing block would break the built-in tools.

## Step 4: memory files

Inspect `<profile>/memories/MEMORY.md` and `<profile>/memories/USER.md` for stale
Caveman defaults, tool instructions, and old setup paths. Repair only relevant
conflicts after the migration. Memories can conflict with prompt guidance; they
do not universally outrank system instructions. Use the profile's supported memory
interface, preserving unrelated preferences and history.

## Step 5: cost settings

Inspect the effective model and provider-specific reasoning settings per requested
profile. Preserve model, effort, and context-window choices unless the user approves
a change. Separate profiles can support an approved cost experiment, but this setup
does not require a cheaper model or reduced effort for any profile.

## Restart

New `hermes` CLI sessions pick up plugin and prompt changes immediately. A running
gateway (`hermes gateway`) may keep loaded hooks. Do not restart it without user
approval; report a pending restart separately from verified fresh-session behavior.

## Verify

Native record: `<profile>/state.db`, SQLite. For the smoke-test session:

- the stored system prompt contains the complete fork, including its last sentence
  (`full and ultra drop articles and conjunctions and are not the default`), not
  only the precedence sentence near the top; check for unwanted RTK instructions;
- configured/session model metadata matches the intended model, and actual
  per-turn author/provider records are checked where exposed; `sessions.model`
  alone is not proof that no fallback handled a turn;
- messages show no unwanted CodeGraph/RTK tool use, respecting a recorded explicit
  CodeGraph exception. Discover the installed database schema before querying.

Check the live MCP inventory for the requested profile; no CodeGraph unless the
user chose the existing-server exception. Test a successful fresh session with the
normal config and harmless tool work; report unexposed runtime fields as unverified.
