# Rationale: why the setup looks the way it does

`SETUP.md` says what to do. This file says why, for each decision, with the
evidence that drove it. Two nine-stage blind multi-agent reviews produced these
conclusions (2026-09-03 on the Caveman suite, 2026-09-04 on CodeGraph, Context
Mode, and RTK), backed by first-hand probes on a working machine. Where a
conclusion rests on that machine's particulars rather than on the tool, this
file says so.

## The standard everything is judged against

Token efficiency is secondary. The primary criteria are: requirements met,
reasoning that looks ahead, research and result quality preserved, verification
done, and no encouragement to stop at "enough" before the task is satisfied.

This ordering came from an observed regression: an agent stack with several
token-saving tools installed began producing work that looked complete, skipped
verification, and stopped early. The investigation that followed asked, of
every installed tool, not "does it save tokens" but "does it change what the
agent reads, verifies, or decides to stop at". Any tool that does is
*process-changing*, and a process-changing tool needs evidence that requirement
coverage holds with it on before it earns always-on status. Byte savings are
not that evidence.

## Why wording compression is kept, and only at lite

Caveman's `lite` level keeps articles and complete sentences; `full` drops
articles and allows fragments; `ultra` strips conjunctions. Upstream's own text
concedes where that goes wrong: its Auto-Clarity section tells the agent to
abandon compression when "compression itself creates technical ambiguity", and
the maintainer advises "Use lite/off for detail-heavy work". Fragments and
dropped conjunctions are where meaning is lost: a negation, a condition, a
unit. Lite removes filler and nothing else, so it is the only level compatible
with the standard.

Two failure modes shaped the rules beyond the level choice. Negations flip
meaning when dropped, so the rules forbid dropping them explicitly. Article
stripping was measured decaying with session length in one 409-session study
(5.42 articles per 100 words at message 1, 9.15 by message 11+), which is why
the rules say "persist across all turns" and why lite, which keeps articles on
purpose, is less exposed to that drift than full.

## Why only four Caveman skills, and not the fifth

The caveman suite ships around twenty skills. Four are wording helpers
(`-commit`, `-review`, `-help`, `-stats`) and change nothing about process.
The rest were excluded because each one changes process in a way the standard
forbids:

- `caveman-explore` delegates to a small model and demands "ONLY an evidence
  block" as the reply.
- `cavecrew` and its presets delegate to `haiku`; the builder preset has no
  shell.
- `caveman-compress` rewrites memory files; upstream says its own checks "do not
  prove semantic equivalence".
- `lean-build`, `verify-and-stop`, `surgical-patch`, `investigate-first`,
  `safe-refactor`, `migration` each add a stop condition or a scope limit.
  `investigate-first` was the closest call; it embeds a model-judged
  "sufficient evidence" stopping rule, which is exactly the judgment the
  standard does not want automated.

The fifth wording skill, `caveman` itself, was installed after the first review
and removed after the second. Its `SKILL.md` body hardcodes `Default: **full**`,
`ACTIVE EVERY RESPONSE`, and `No preamble, plan, or progress note`. All three
contradict the setup. A skill body loads when the agent judges its description
to match, so the defaults it carries are both wrong and unpredictable. The lite
wording rules belong in the always-on rule file, where they apply every time
and can be read and audited.

**Why the rules live in one place.** An earlier revision of the setup installed
the wording rules twice: once as a block in the agent's rule file and once as the
fork's "Output style" section, both into the same file. Two near-copies cost
input tokens every turn and diverge the first time one is edited. The fork's
section is now the only carrier. It also says the mode is on from the first
reply with no invocation, because the earlier wording only pinned the level and
could be read as "if caveman is invoked, use lite"; a reviewer on a second
machine read it that way.

**Why the caveman config file is not the switch.** Caveman's plugin configuration
reader resolves `defaultMode` from the environment, repository, and user config;
the user path honors `XDG_CONFIG_HOME`, otherwise Windows `APPDATA` or
`~/.config`.[2] That mechanism does not activate the fork's lite rules in the
four-skill setup. Its presence invited false verification: "the level is lite,
I checked the config". Creation was removed. An existing file is retired only
after backup, with confirmed setup origin, only the old `defaultMode: lite`
setting, and no enabled Caveman integration using it. Unknown consumers or
provenance mean preserve and report, not deletion across another agent's setup.

**Why the installer is not used.** `npx skills add JuliusBrussee/caveman` has no
ref argument and resolves to the repository head. On one machine it had
installed `caveman`, `cavecrew`, `caveman-compress`, and `lean-build` alongside
the intended four, which is the exact set the standard excludes. A pinned clone
plus copying four directories is longer to type and has no such failure mode.
A model executing the setup also declined to run the installer on its own
judgment (a third-party script from the internet), so the copy route is the one
that gets done.

## Why the Karpathy guidelines are forked

The upstream file is good and the fork keeps its three core principles nearly
verbatim. The changes are:

**Dropped: "Surgical Changes."** Upstream tells the agent to leave adjacent
problems alone. That pulls in the same direction as the excluded scope-limiting
skills. An agent that notices a related defect should say so, not be instructed
to ignore it.

**Trimmed: "Simplicity First."** Reduced to its lead-in and the senior-engineer
test. The fuller upstream version leans toward doing less, which again overlaps
with scope narrowing.

**Added: precedence over output-style rules.** Caveman says "No preamble, plan,
or progress note before or between calls." Karpathy says "state a brief plan,
each step paired with its verification check." Left unresolved, the agent picks
one silently, and in practice picks the one that produces less text. The fork
says the guidelines win: a brief plan is not narration. This single sentence is
what stops the no-look-ahead failure mode, and it is the most important line
in the setup.

**Added: precedence over tool instructions that narrow evidence.** The second
review found that retrieval tools inject standing text like "don't re-verify
them with grep", "treat each block as a Read you have already performed", and
"return: file path + 1-line description". The original precedence sentence only
covered *style* rules, so these evidence rules had no counterweight and the
model resolved the conflict on its own. The fork now names the pattern
(not-re-verify, not-read, already-read, stop-after-N-calls, path-only reply) and
states that retrieval and compression tools are leads, not verification.
This is written generically so the next tool cannot reopen the gap.

**Added: scope for "ask".** Karpathy says "If something is unclear, stop. Ask."
A separate operating rule for unattended batches says "never stall". Both are
correct in context. The fork resolves them: when the user has explicitly handed
over a batch and left, decide, log each decision with its reason, and report;
everywhere else the ask rule stands.

**Added: evidence is never compressed.** The precedence paragraphs protect
plans, assumptions, and hedges. They did not protect evidence, and caveman's
own rules ask for the "shortest decisive line" of an error rather than the
error. In debugging the decisive line is often not identifiable in advance, so
that rule is a correctness hazard exactly where the user most needs to judge
for themselves. The fork now states that error output, stack traces, diffs,
test failures, and numbers are reproduced in full, or the omission is stated.
This came from a review of the setup on a second machine, not from the
original council, and it is the one addition that changes what the agent shows
rather than how it phrases things.

**Why the upstream plugin is disabled where the fork is installed.** Leaving
`andrej-karpathy-skills` enabled beside the fork loads the same guidance twice:
once always-on, once on a coding-scoped trigger, in the unforked form that
still carries "Surgical Changes". The per-agent notes disable it.

**Delivery: always-on, never as a skill.** Upstream ships the guidelines as a
skill whose description is coding-scoped ("Use when writing, reviewing, or
refactoring code"). That description does not fire on research, planning, or
diagnosis, which is exactly where the second review found the quality risk
concentrated. Karpathy already prescribes an independent oracle (deterministic
tests) for bug fixes and refactors; the unprotected phases are evidence
gathering, and a coding-scoped skill leaves them unprotected.

## Why CodeGraph is off as an ambient integration

CodeGraph builds a per-repository call graph and exposes one tool that returns
graph-selected source. Its standing instructions, injected into every session
via MCP initialization text and generated rule-file blocks, say:

> Trust codegraph's results, don't re-verify them with grep

> Treat each block as a Read you have already performed: do not Read a file
> shown here

> ONE call usually answers the whole question

Its instruction template states a call budget that nothing enforces, and the
vendor's own design document names the goal as making the agent stop reading.
That is the standard's failure mode, written as an instruction.

Two behaviours were reproduced first-hand on the pinned version and on the next
release. Asked about a symbol name that does not exist, the `callers`,
`callees`, and `impact` paths returned results for the nearest match with no
absence statement: confident wrongness rather than "not found". The default
`explore` path returned related source without stating that the exact symbol
was absent. Exact-name queries, by contrast, were correct with no false
positives, and returned source was byte-identical to disk. So the tool is
accurate when the question is well-formed and misleading when it is not, and
its instructions tell the agent not to check which case it is in.

The evidence for savings is real but regime-bound. The vendor's own A/B matrix
(one run per arm, which it calls "directional") shows fewer reads and calls on
repositories in the hundreds-to-thousands of files, and ties or slower results
on small ones. Output caps are tightest in the under-150-file tier, where
relationship edges and the completeness signal are off by default. The machine
this was measured on had indexes of 2, 2, and 114 files. In that regime the
vendor's evidence shows no gain and the instruction text still applies, so
turning it off costs nothing.

**On a machine with large repositories the calculus is different**, and
`SETUP.md` carves out that case: keep the MCP server so the tool is available
by name, remove the always-on block and hook, and add a rule line that its
output is a lead to be read, not verification. Whether the always-on block
should return on such a machine is an empirical question the reversible
experiment below is designed to answer; the instruction text does not become
acceptable just because the retrieval is faster.

## Why Context Mode is off

Context Mode indexes large tool outputs and returns intent-matched excerpts. Its
routing block, marked MANDATORY and injected by lifecycle hooks into every
session and every subagent prompt, says:

> Do NOT read raw data into context. PROGRAM the analysis, not COMPUTE it.

> ONE call replaces 30+.

> Write artifacts to FILES - never inline. Return: file path + 1-line
> description.

Its benchmark is 21 deterministic fixtures measuring bytes. There is no
end-to-end task study in either direction. Among the three tools it is the one
with the least evidence about task quality.

Two behaviours were reproduced first-hand. A 50 KB output with one decisive line
unrelated to the stated query intent returned a 1.8 KB excerpt without that
line; a later targeted search for the exact string did retrieve it. Omitted
material is retained and recoverable, but only if the agent knows to search
again, and its search throttles after three calls. "Announced and recoverable"
at the tool's layer is silent at the agent's layer, which is what a user
perceives as no look-ahead.

The second finding was discovered because it broke the review's own controls. A
Claude session launched with built-in tools restricted to `Read` still executed
shell commands through Context Mode's `ctx_execute` MCP tool. The `--tools` flag
governs built-in tools only; an MCP server with an execution tool restores a
capability the invocation intended to withhold. That is not a bug in Context
Mode, but it means any reliance on tool allowlists is undone by installing it.

Three open issues confirmed on the pinned version reinforced the decision:
permission anchors `//` and `~/` never match so its deny gate fails open for
them; content databases idle for an hour are deleted on the next session start,
reproduced with data loss; stale cross-session memory can rank above fresh
same-session captures.

## Why RTK is off, and why it is the mildest case

RTK rewrites shell commands through a filtering proxy, transparently: the agent
never sees the rewrite. Unlike the other two, it carries no anti-verification
text. Its filtering is announced in output ("480 matches in 12 files, showing
200") with recovery commands, its contributor policy forbids silent caps, and
first-hand probes on the pinned version confirmed both: grep truncation was
announced with working recovery commands, and `rtk read` was byte-identical to
`cat` on a 1,200-line file. All three of its adapters fail open. It is a
different category from the two above.

It is off anyway, for three reasons.

**The savings claim does not survive its own documentation.** The README says
60-90% token reduction. The docs say bash output is "the only thing RTK
controls", "one contributor to input tokens", and that "a command showing 90%
fewer output bytes does not make your session 90% cheaper". It ships no
tokenizer; `rtk gain` estimates at bytes/4. On the machine measured, its own
estimator reported 5.8% saved over 12,365 commands.

**Independent measurement is neutral to negative.** JetBrains, on Claude Code
with SkillsBench: +7.6% cost at low reasoning effort (p=0.004), plus or minus 0% at high
effort, task quality unchanged. A maintainer attributed the cost to extra API
turns when a task is not well covered by the filters. A separate 35-case CI-log
study found downstream diagnosis quality degraded under `rtk-log` with a 13.3%
confident-error rate, recovering only through additional tool calls. Both
studies state their own caveats; neither shows a gain.

**Its transparency creates a gate hazard.** A pattern gate that inspects the
command string sees `rtk git push` rather than `git push`. The Claude hook's
documented response also carries a `permissionDecision` field. Whether approval
systems evaluate the pre-rewrite or post-rewrite command, and whether that
field overrides the user's rules, was not established. Until it is, a
transparent rewrite sits between the agent and every safety gate.

RTK is the likeliest of the three to return, because its hooks are separable
from its text and it fails open. The conditions are in the experiment below.

## Why Ponytail is not installed

Ponytail's skill asks the agent to decide whether a task "needs to exist at all"
and to skip speculative need, choosing "the laziest solution that actually
works". That is a scope-narrowing stop condition of the same family as the
excluded caveman skills, and it was removed on the same grounds during the first
review. It was never wired into a session on this machine after that.

## Why the other installed skills stayed

The second review audited 45 locally installed skills beyond the caveman set
and kept all of them. None is always-on; each loads only when the agent judges
its description to match. Their direction is mostly toward more verification and
look-ahead, "look at X before building around 'X is not possible'", "a
marker-only probe does not validate tools", "implementation-lean output modes
must not govern design work". The few that narrow scope are bound to a named
risky task (device takeover, outbound messaging, destructive database work) and
their gates were kept exactly as written. "Quality" does not authorize bypassing
a safety gate.

## What was learned about the review method itself

Several findings came from the harness and from running the setup on a second
machine rather than from the subjects, and each changed the setup.

**A clean exit code is not evidence that the requested model answered.** The
provider silently substituted a different model mid-run after a safety
classifier flagged the packet text. The CLI exited 0 with a normal completion
status and no error; only the native session record showed the change. Every
verification step now reads the native record's model field on every turn.

**Config read-back is not verification.** `--settings` overrides removed hook
*decisions* but not hook *injection*; only an alternative configuration
directory zeroed both. A seat evaluating Context Mode was receiving Context
Mode's own mandatory instructions until that isolation was built. Every
verification step now runs a fresh session and inspects what actually reached
the model.

**One agent's config can be another agent's input.** Cursor loads hooks from
`~/.claude/settings.json` and runs them, merging them below its own. This was
verified, not just read: a `PreToolUse` marker hook placed only in the Claude
file fired during a shell call in the Cursor CLI, and again in a fresh Cursor
IDE install, with no Cursor setting touched in either case and the IDE's own
Hooks tab reporting none configured. The documentation describes this as an
opt-in; on the builds tested it was the default. A hook removed from Cursor's
`hooks.json` but left in Claude Code's settings is still live in Cursor, and
Cursor's UI will not show it. The same shape appeared on the reviewed machine
in a different pair: Claude Code's rule file had leaked into a Hermes profile
through a home-rooted path. This is why the Cursor notes inspect inherited Claude
configuration and why the verification step reads what reached the
model rather than which file was edited or what a settings screen reports.

**Plugin records can disagree.** On an observed Claude Code installation,
`enabledPlugins` appeared in both `settings.json` and
`plugins/installed_plugins.json`; on that machine
Context Mode was `false` in the first and `true` in the second, with the
marketplace still listed. Which layer wins at
runtime was not established by a controlled test (the plugin's skills were
absent from the session, which suggests the settings file won). This is not a
universal registry schema or proof of later re-enablement. The setup uses supported
disable/list commands, reconciles existing conflicting flags, and leaves an
inventory-only record alone instead of inventing enablement keys.

**Cost settings can also change capability.** On the second machine the
settings selected a 1M-context model with elevated reasoning effort for every
session. This did not establish which cost line dominated or prove that lowering
effort preserved quality. Current Claude documentation distinguishes actual token
usage, pricing, and available capacity: its 1M window uses standard model pricing
without a premium beyond 200K, while subscription coverage and usage-credit
requirements vary by model and plan.[1] A larger window
can retain more billable context before compaction, but an unused window is not a
flat per-turn charge. Reducing it can force earlier compaction and lose needed
context. The setup reports model, effort, and window choices and preserves them
unless the user approves a named setting and value. The fork cannot guarantee
equal quality at lower effort. CLI launch overrides support a temporary trial;
interactive model/effort commands can persist defaults and need care.[1]

**A setup that does not measure cannot claim savings.** Nothing in the earlier
verify section measured tokens or quality; both checks concerned rule loading.
The baseline must be captured before the first edit, using five fixed tasks with
written quality criteria and comparable model, effort, window, fixtures, and cache
conditions. Native input/output, cache-read/cache-write, and reasoning categories
are used where exposed, without double-counting. A missing baseline means
unmeasured, not reconstructed savings. Five tasks test a local workload, not the
universal claim of token savings without quality loss.

**Self-report is a smoke test, not evidence.** The earlier verify section said
not to ask the model about itself and then did so twice. Asking is kept as the
quick check; the evidence is the agent's MCP list and its native session
record. A harmless shell command and file read exercise tool hooks that a text-only
question would miss. No calls alone does not prove no tools were available; an
OAuth failure before assistant work proves neither loading nor absence. Inspect
per-turn assistant model IDs and resolve aliases instead of requiring one usage
summary key: helper-model accounting is not necessarily main-answer fallback.

**A model may decline part of the setup.** In an earlier Windows run,
the model reported completing config edits and
refused the one step that ran a third-party installer, on its own judgment,
and said so. Another run had stopped partway without saying which
steps were skipped. The setup now avoids installers entirely (clone at the
pin, copy four directories) and the per-agent notes tell the executing agent
to name any step it declines rather than skip it silently.

## What was and was not tested

Earlier deployments were probed on Claude Code, Codex, Grok CLI, Hermes, the
Cursor CLI, and the Cursor desktop IDE, using fresh sessions and native records
where available. Those historical probes do not certify every later revision of
these installation notes or establish end-to-end quality and cost improvements.

**Latest Windows follow-up (2026-09-10).** User-supplied screenshots report the
updated setup applied, the four copied helpers plus an unrelated `find-skills`
retained, stale memories repaired, and a legacy Caveman config left in place.
They also report changes from `opus[1m]` to `opus` and lowered effort. Runtime
checks did not reach assistant work because OAuth expired; there was no pre-change
benchmark. This is evidence about the reported deployment and its blocked checks,
not independent access to the Windows filesystem or proof of savings. It prompted
the before-edit baseline, conditional cleanup, closing memory repair, and explicit
cost-setting consent in the revised instructions. Two reported typos were absent
from the current repository and were not edited there.

**Hermes template test.** On Hermes v0.20.4, local source `ad1ee7a3`, the old
`max_chars=6000` example raised `ValueError: system prompt section max_chars must be between 1 and 4000`.
An isolated test of the corrected `4000` example registered
and rendered the entire 3789-character ASCII fork (3788 after trimming its final
newline). Source inspection shows overlong content is skipped, not truncated.
The verification therefore checks the full fork, not only its first sentence.
The plugin API page[3] was consulted; the cap was established from installed source.

**Instruction-file preservation.** Existing unrelated global rules must not be
replaced by a one-line import. Claude's import can be added once; Codex's current
documented discovery reads the first nonempty global `AGENTS.override.md` or
`AGENTS.md`.[4] Its notes now embed the actual guidance in that active file rather
than treating an unverified `@path` reference as proof of automatic loading.

For Cursor, three documentation claims were tested directly. Two held: a plain
`.md` in `.cursor/rules/` is ignored while the same content as an
`alwaysApply: true` `.mdc` loads, and Claude Code hooks fire inside Cursor.
One did not hold as written: the third-party-config opt-in was not required
for Claude hooks on either the CLI or the IDE. The IDE probe also showed the
fork's brief-plan rule in effect unprompted: the agent opened with "Plan: ...
Verification: shell output plus tool catalog" before answering.

One thing was not tested: none of the three tools had been installed into
Cursor on this machine, so their removal steps are from each tool's own install
instructions rather than from undoing a real install.

The tested free Cursor plan could not pin a model; the answering model is recorded in the
native chat store and was `cursor-grok-4.5-high` for the CLI probes. The IDE
showed `Cursor Grok 4.6 Medium` in its composer for its probe.

## The reversible experiment

A tool returns to always-on status on an agent only after this passes on that
agent. It is read-only: throwaway working and configuration directories, no
persistent change.

**Arms.** Intact current configuration; an isolated configuration with no tool
integrations; and one arm per tool with only that tool added to the isolated
base.

**Tasks.** Three shapes, each with a written requirement list: a coding task
with a breaking change two files from the obvious edit; a research task on a
repository containing one exact symbol and one plausible-but-nonexistent name;
a verification task whose log has one failure, one skip, one strict xfail, one
decisive line past any per-file cap, and a nonzero exit.

**Runs.** Three per task per arm. Keep main-answer models identical across arms,
checking native assistant attribution and resolving aliases. Separate helper usage
from main-model fallback. Exclude main-model substitutions from the comparison;
unavailable authorship metadata makes the model-controlled result unverified.

**Scoring.** Requirement coverage against the written list with a four-state
audit so "proven manually" never scores as shipped; a separate confident-wrong
count for any asserted fact the fixture contradicts; deterministic tests for the
coding task; blinded scoring of the research task.

**Pass.** Every stated requirement met, zero confident-wrong assertions, tests
pass, skip and xfail reported accurately, the decisive line present in the reply
or its artifact.

**Interpretation.** If a single-tool arm and the isolated arm pass equally, that
tool's return is evidence-backed on that agent. If the intact arm fails where
the isolated arm passes, the stack is implicated regardless of which single arm
fails.

## What is not claimed

The original regression's cause was never established. Its agent, model, task,
and active integrations were unknown; the review worked from the instruction
text and first-hand probes, and its verdict on that text stands whether or not
any of the three tools was involved in that specific incident. The
recommendation to turn them off rests on what they instruct the agent to do,
not on proof that they did it.

## Sources

Vendor pages and the pinned source below were consulted on 2026-09-10.
Version-dependent claims must be rechecked against the installed agent and current docs.

[1] https://code.claude.com/docs/en/model-config
[2] https://raw.githubusercontent.com/JuliusBrussee/caveman/3b74643f4d910f496babd4e634b1ba7168816f14/src/hooks/caveman-config.js
[3] https://hermes-agent.nousresearch.com/docs/developer-guide/plugins
[4] https://developers.openai.com/codex/guides/agents-md
