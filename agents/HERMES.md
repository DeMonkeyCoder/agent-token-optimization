# Hermes

Applies the base setup. This file says only where things live in Hermes and what is
specific to it. Everything below is per profile: the default profile is
`~/.hermes`, named profiles are `~/.hermes/profiles/<name>`, and each has its own
`config.yaml`, prompt, plugins, skills, and memories. Repeat every step for every
profile, and use `hermes --profile <name>` for the commands.

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
        max_chars=6000,
    )
```

Paste the fork text verbatim inside the triple quotes. Then:

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

`<profile>/memories/MEMORY.md` and `<profile>/memories/USER.md` are injected into
every turn and outrank the prompt in the model's reading. Search both for `caveman`,
`rtk`, and `codegraph`. A line saying caveman is full, or telling the agent to
prefix commands with rtk, overrides everything above.

## Step 5: cost settings

`config.yaml` holds `model` and, depending on provider, a reasoning or effort
setting. Set them per profile; a profile used for mechanical batch work should not
share the design profile's model.

## Restart

New `hermes` CLI sessions pick up plugin and prompt changes immediately. A running
gateway (`hermes gateway`) keeps loaded hooks until `hermes gateway restart`, run
from a plain shell outside any agent session.

## Verify

Native record: `<profile>/state.db`, SQLite. For the smoke-test session:

- `sessions.system_prompt` (or the `system_prompts` row its hash points at) contains
  the fork's precedence sentence and no `rtk` instruction line;
- `sessions.model` is the model you configured;
- `messages` for the session has no `tool_name` from codegraph or rtk.

`hermes --profile <name> mcp list` should not show `codegraph`.
