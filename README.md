# agent-token-optimization

Token-efficient output from AI coding agents, without paying for it in result
quality. Agent-agnostic; tested on Claude Code, Codex, Grok CLI, Hermes, and Cursor
(CLI and desktop IDE).

How to use it: give an agent `SETUP.md` plus the `agents/` file for each agent you
want configured, in one prompt. `SETUP.md` is the shared base; the per-agent files
say only where things live and what is specific to that agent. Neither refers to
the other by name, so any subset can be pasted together.

| File | What it is |
|---|---|
| `SETUP.md` | The base: what to do. Embeds the forked guidelines, so it needs nothing else. |
| `agents/CLAUDE-CODE.md` | Claude Code paths, plugin registries, hooks, MCP, cost settings |
| `agents/CODEX.md` | Codex CLI paths, `config.toml` tables, `hooks.json` |
| `agents/CURSOR.md` | Cursor CLI and desktop: `.mdc` rules, `mcp.json`, `hooks.json`, and the Claude hook leak |
| `agents/GROK.md` | Grok CLI rules directory, `config.toml`, MCP flags |
| `agents/HERMES.md` | Hermes per-profile plugin, prompt, skills, memories, gateway restart |
| `RATIONALE.md` | Why. The evidence behind each decision, what was measured on which machine, and what is machine-dependent. |
| `karpathy-guidelines-fork.md` | The always-on guidelines file, byte-identical to the copy embedded in `SETUP.md` |
| `karpathy-guidelines-fork.patch` | Reproduces the fork from upstream: `git checkout 2c60614 && patch -p1 < karpathy-guidelines-fork.patch` |

All files are plain ASCII: no em dashes, arrows, curly quotes, or other characters
a keyboard cannot produce. Keep it that way when editing. The one exception is the
removed (`-`) lines in `karpathy-guidelines-fork.patch`, which quote upstream
verbatim.

## Tools covered

| Tool | Source | Verdict |
|---|---|---|
| Caveman | [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman) `3b74643` | Four wording skills; the `caveman` skill body and every process-changing skill excluded; lite rules carried by the fork, not by a skill or a config file |
| Karpathy guidelines | [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills) `2c60614` | Forked, always-on, outranks compression and tool text, protects evidence from compression |
| Ponytail | [DietrichGebert/ponytail](https://github.com/DietrichGebert/ponytail) | Not installed: scope-narrowing stop condition |
| CodeGraph | [colbymchenry/codegraph](https://github.com/colbymchenry/codegraph) `v1.5.0` | Off as ambient; binary and indexes kept. Regime-dependent: see `RATIONALE.md` for large repositories |
| Context Mode | [mksglu/context-mode](https://github.com/mksglu/context-mode) `v1.0.169` | Off as ambient; stores kept |
| RTK | [rtk-ai/rtk](https://github.com/rtk-ai/rtk) `v0.45.0` | Off as ambient; binary kept. Mildest case; likeliest to return after the experiment in `RATIONALE.md` |

Every verdict is against the pinned version named. All six upstreams move; re-read
before adopting a newer one.

## Provenance

Two nine-stage blind multi-agent reviews (2026-09-03 on the Caveman suite,
2026-09-04 on CodeGraph, Context Mode, and RTK), each reading upstream source at the
pinned commit, the installed files on a working machine, vendor benchmarks, issue
threads, and first-hand throwaway-fixture probes of each tool's failure and recovery
behaviour. A later run of the setup on a second machine (Windows, Claude Code only)
produced the review that led to the single-carrier rules, the evidence clause, the
cost-settings step, and the measurement step; `RATIONALE.md` records which findings
came from where.
