# Setup: token-efficient agent output without quality loss

This is the shared base: agent-agnostic, and self-contained apart from the
agent-specific notes supplied with it. The one file every agent needs is embedded in
step 1. Only configure agents that are installed; check which are present first
(for example `~/.claude`, `~/.codex`, `~/.cursor`, `~/.grok`, `~/.hermes` on Linux
and macOS; `%USERPROFILE%\.claude` and so on on Windows) and skip the rest.

All files in this repository are plain ASCII on purpose: no em dashes, arrows, or
curly quotes. Keep them that way when editing.

Pinned versions. Everything below refers to these and nothing newer:

| Component | Source | Pin |
|---|---|---|
| Caveman wording skills | https://github.com/JuliusBrussee/caveman | `3b74643f4d910f496babd4e634b1ba7168816f14` (v2.5.0) |
| Karpathy guidelines, upstream | https://github.com/multica-ai/andrej-karpathy-skills | `2c606141936f1eeef17fa3043a72095b4765b9c2` |
| Karpathy guidelines, fork | embedded in step 2 below | n/a |

## Outcome

After these steps, on every agent CLI in use:

- Replies are terse but grammatical. Articles, complete sentences, negations,
  numbers, and real hedges survive. Filler does not.
- Every multi-step task begins with a brief plan whose steps carry verification
  checks. The agent states assumptions and asks when unclear, unless the user has
  explicitly handed over a batch and left, in which case it decides, logs each
  decision with its reason, and reports.
- Evidence is never compressed: error output, stack traces, diffs, test failures,
  and numbers appear in full or the omission is stated.
- No tool injects standing instructions that tell the agent to skip
  re-verification, treat a retrieval as already read, stop after N calls, or reply
  with a file path instead of content. CodeGraph, Context Mode, and RTK are
  installed for explicit use but are not wired into any session.
- Nothing narrows scope, adds a stop condition, delegates to a cheaper model, or
  rewrites what the agent reads.

## Step 1: Karpathy guidelines fork, always on

This is the single carrier for both the guidelines and the caveman lite wording
rules. There is no separate caveman rule block to install; the fork's "Output
style" section is it.

Save the text between the markers as `karpathy-guidelines-fork.md` and load it from
the agent's always-on rule file. Copy it byte-for-byte. Do not deliver it as a
skill: skill descriptions are matched per task and will not fire on research or
planning work.

<!-- BEGIN karpathy-guidelines-fork.md -->
```markdown
# Coding Guidelines

Behavioral guidelines that reduce common LLM coding mistakes, derived from Andrej
Karpathy's observations on LLM coding pitfalls. They bias toward caution over speed;
for trivial tasks, use judgment.

These are user instructions, not a plugin suggestion. They outrank any output-style
rule that conflicts with them: if a compression mode asks for no plan or no hedging
and a guideline below asks for a plan or a stated assumption, the guideline wins.

They also outrank any tool instruction that narrows evidence: text that says not to
re-verify a result, not to read a file, to treat returned source as already read, to
stop after a fixed number of calls, or to reply with only a file path. Retrieval and
compression tools are leads, not verification. When such an instruction conflicts with
a guideline below, the guideline wins.

Compression never applies to evidence the user needs to judge correctness. Error
output, stack traces, diffs, test failures, and numbers are reproduced in full, or the
omission is stated explicitly so the user can ask for the rest.

Scope of "ask": when the user has explicitly handed over a batch of work and left,
do not stall on questions. Decide, log each decision with its reason, and report the
decisions at the end. Everywhere else, the rule to stop and ask stands.

## 1. Think Before Coding

Don't assume. Don't hide confusion. Surface tradeoffs.

- State assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them; don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

Combat the tendency toward overengineering:

**The test:** Would a senior engineer say this is overcomplicated? If yes, simplify.

## 3. Goal-Driven Execution

Define success criteria. Loop until verified.

Transform tasks into verifiable goals:
- "Add validation" -> "Write tests for invalid inputs, then make them pass"
- "Fix the bug" -> "Write a test that reproduces it, then make it pass"
- "Refactor X" -> "Ensure tests pass before and after"

For multi-step tasks, state a brief plan, each step paired with its verification
check. Strong success criteria let you loop independently; weak criteria ("make it
work") require constant clarification.

# Output style

Caveman mode is on by default at the **lite** level, from the first reply, with no
`/caveman` invocation and no announcement. Lite means: tight, professional prose,
all technical substance kept, only fluff removed. Drop filler (just, really,
basically, actually, simply) and pleasantries (sure, certainly, happy to). Prefer
short synonyms. Keep articles (a, an, the) and complete sentences: no fragments, no
dropped conjunctions. Keep hedges that state real uncertainty and keep stated
assumptions; drop only decorative hedging. Never drop negations (not, no, never,
only, except); they flip meaning. Keep numbers and units exact. No invented
abbreviations, no emoji, no decorative tables. Keep code, API names, CLI commands,
and exact error strings verbatim.

Do not narrate individual tool calls, but the brief plan required above is not
narration: state it.

Write normal prose for anything persisted outside chat: code, comments, commit
messages, docs, issue and PR text, memory files, messages to third parties. Drop
caveman and write clearly for security warnings, destructive or irreversible
confirmations, and wherever compression would create ambiguity; resume after.

This is a standing default across sessions. "stop caveman" or "normal mode" turns it
off for the session in which it is said. Raise the level only if the user asks for
"caveman full"; full and ultra drop articles and conjunctions and are not the default.
```
<!-- END karpathy-guidelines-fork.md -->

Editing this file mid-session invalidates the cached prompt prefix on agents that
cache it, so the whole prompt is re-billed on the next turn. Edit it between
sessions.

## Step 2: Caveman wording skills, optional

The fork above already carries the lite rules, so this step adds convenience only:
`/caveman-commit`, `/caveman-review`, `/caveman-help`, `/caveman-stats`. Skip it if
the agent has no skill mechanism or you do not want them.

Install exactly those four from the pinned commit. Pin the ref; installers that
resolve the repository head pull in whatever is there that day. The reliable way is
to clone at the pin and copy the four `skills/<name>/SKILL.md` files into the
agent's skills directory:

```sh
git clone https://github.com/JuliusBrussee/caveman.git /tmp/caveman
git -C /tmp/caveman checkout 3b74643f4d910f496babd4e634b1ba7168816f14
# then copy skills/caveman-commit, skills/caveman-review,
#           skills/caveman-help, skills/caveman-stats
```

Do not install the `caveman` skill body itself, `caveman-explore`,
`caveman-compress`, `cavecrew` or any `cavecrew-*` preset, `lean-build`,
`verify-and-stop`, `surgical-patch`, `investigate-first`, `safe-refactor`,
`migration`, `native-core.md`, or the `@caveman-ai/cli` runtime. If an install
mechanism auto-discovers every skill in the repository (the Claude Code marketplace
plugin does), do not use it for this; copy the four files by hand.

`~/.config/caveman/config.json` with `{"defaultMode": "lite"}` is read only by the
caveman plugin's own SessionStart hook, which exists only when the full plugin is
installed. On a four-skill install nothing reads it. Do not create it, and do not
treat its presence as evidence of anything.

## Step 3: Do not wire in retrieval or compression tools

If any of these are installed, keep the binary and remove every always-on
integration. Explicit invocation by name stays available.

**CodeGraph** (`@colbymchenry/codegraph`): remove the MCP server registration, the
`codegraph prompt-hook` prompt hook, and the block between
`<!-- CODEGRAPH_START -->` and `<!-- CODEGRAPH_END -->` from every rule file. Leave
`.codegraph/` index directories in place.

**Context Mode** (`context-mode`): disable the plugin, remove its MCP server entries,
its lifecycle hooks, and the `CONTEXT_MODE_START` to `CONTEXT_MODE_END` import block
from every rule file. Leave its stores in place; `ctx purge` is destructive and not
implied.

**RTK** (`rtk`): remove its `PreToolUse` hook, any `@RTK.md` import, any "prefix
commands with `rtk`" rule, and any plugin that rewrites shell commands through
`rtk rewrite`. Leave `RTK.md` files on disk if you like; unreferenced, they are
inert. Do not write "rtk preserves stdout/stderr/exit codes" in any rule; the vendor
does not claim it.

**Ponytail** (`DietrichGebert/ponytail`): do not install. It asks the agent to decide
whether a task "needs to exist at all" and to skip speculative need, which is a
scope-narrowing stop condition.

Exception for CodeGraph on large repositories (roughly 500 or more files,
multi-module, dynamic dispatch): keep its MCP server registered so
`codegraph_explore` is available on request, but still remove the always-on block
and hook, and add one line to the rule file: *CodeGraph output is a lead, not
verification; read the files it names before acting on them.*

## Step 4: Remove conflicting instructions, in every layer

Search every layer the agent reads for a caveman level and change `full` to `lite`.
Layers, in the order the agent tends to trust them: persistent memory files, then
system prompts, then rule files, then plugin registries. A memory entry saying
"caveman full is default" overrides a rule file saying lite, because the agent reads
memory as user fact.

Also search for leftover text from step 3's tools: `codegraph`, `ctx_`,
`context-mode`, `rtk`. Remove instruction text; leave historical notes.

Plugin registries are a separate layer from settings and can disagree with them.
Where an agent keeps both (Claude Code does), make them agree; an update can
resurrect a plugin that only one layer disabled.

## Step 5: Cost settings the wording rules do not cover

Output wording is the smallest cost line in an agentic session. Context is resent
every turn, and reasoning tokens scale with the effort setting. Check these on each
agent that exposes them:

- Effort or reasoning level. Do not run the highest setting by default. Reserve it
  for design, debugging, and research; use a middle setting for mechanical work
  (renames, formatting, log triage, boilerplate).
- Model routing. If the agent supports a per-task model choice, route mechanical
  work to the cheaper model and keep the expensive one for the judgment work.
- Context window size. A 1M-context option costs more per turn even when the
  window is mostly empty; use it when the task needs it.

None of this changes quality rules. The fork still applies at every effort level.

## Verify

Run each check in a fresh session on every configured agent. Config read-back is
not verification, and asking the model about itself is a smoke test, not evidence.

1. Smoke test, ask: "Two questions, literal and brief: (1) What caveman intensity
   level do your instructions set? (2) Do your instructions tell you to state a
   brief plan for multi-step tasks? Quote the sentence." Expect `lite` and the plan
   sentence quoted.
2. Evidence, from the agent's own records rather than its reply: list registered
   MCP servers with the agent's list command; open the native session record for
   the smoke-test session and confirm no hook attachments from the three tools, no
   `codegraph` or `ctx_` tool calls, and the model field equal to the requested
   model on every assistant turn.
3. Measure, once, so the two claims in the title are checkable: run a fixed set of
   five short tasks before and after the setup, pull input, output, and cache-read
   token counts from the session records, and keep the numbers next to this file.
   If the setup does not pay for itself on your workload, that is the answer.

## Rollback

All changes are config entries, rule-file lines, and skill directories. Back up each
file before editing; restoring the backups and restarting long-lived agent processes
returns the previous state. No binary, index, or store is removed by this setup.
