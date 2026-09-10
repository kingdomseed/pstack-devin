# pstack → Devin porting contract

Source: `cursor/plugins` `pstack/` @ 0.15.1 (commit `f8abedd`), copied into `plugins/pstack/`.
Style: **full conversion** — every file speaks Devin natively. No adapter skill. Strip all Cursor spellings; do not leave "Cursor does X → Devin does Y" translation tables inside shipped skills (this file is where that knowledge lives).

## Primitive map (Cursor → Devin)

| Cursor | Devin |
|---|---|
| `Task` tool, `subagent_type: generalPurpose` | `run_subagent` tool |
| `subagent_type: generalPurpose` + `readonly: true` | profile `subagent_explore` (read-only, cheap) |
| `subagent_type: generalPurpose` (writes) | profile `subagent_general` (inherits parent model) — or `pstack:worker`/`pstack:sol` when a role names a model |
| `subagent_type: "poteto-agent"` | profile `pstack:poteto-agent` |
| `subagent_type: "Comment Sicko"` | profile `pstack:comment-sicko` |
| `model: <cursor-slug>` on a spawn | `model:` in `agents/*.md` frontmatter, or skill `model:` field. No per-spawn model names — profiles pin models |
| role "inherit-parent" / "auto" | profile `subagent_general` (custom profiles without `model:` get SWE-1.6, NOT the parent — never rely on that) |
| `run_in_background: true` | `run_subagent` background mode (unapproved tools auto-deny) |
| `environment: "cloud"`, `cloud_base_branch`, Cursor cloud agents | Devin cloud session via `/handoff`, or `devin cloud`; swarm cloud workers → Devin cloud sessions |
| `~/.cursor/rules/pstack-models.mdc` | `~/.devin/rules/pstack-models.md` (always-on rule, Windsurf-style `trigger: always_on`) |
| `.cursor/skills/`, `~/.cursor/skills/` | `.devin/skills/` (project), `~/.config/devin/skills/` (global) — `.agents/skills/` also works |
| `~/.cursor/projects/<slug>/agent-transcripts/` | `~/.local/share/devin/cli/transcripts/*.json` and `~/.local/share/devin/cli/summaries/history_*.md` |
| `disable-model-invocation: true` frontmatter | `triggers: [user]` |
| `AskQuestion` tool | `ask_user_question` tool |
| Bugbot, `bugbot[bot]` review comments | Devin Review (app.devin.ai/review, `npx devin-review <pr>`, Bug Catcher, auto-review, autofix). Detection: authors `devin`, `devin-ai-integration[bot]`, `devin-review` |
| Cursor Automations | Devin Automations — webapp app.devin.ai → Settings → Automations, or v3 API `POST /v3/organizations/{org}/automations`/`schedules`. Triggers: schedule (cron), Slack, GitHub, Linear, webhook. Actions: start session, message session, triage monitor, email. NOT plugin-installable — skills ship instructions + prompt templates |
| `cursor-team-kit` `deslop` | bundled at `skills/deslop` — references stay, path is in-plugin |
| `cursor-team-kit` `control-cli`/`control-ui` | drop or map to `devin` CLI; do not reference cursor-team-kit |
| Cursor built-in `create-skill` | write SKILL.md files directly per Devin's creating-skills format (frontmatter: name, description, argument-hint, model, subagent, agent, allowed-tools, permissions, triggers) |
| `/loop` | `/loop` exists in Devin (prompt → auto-review diff loop). Keep references |
| `/goal` markers, plan files | Devin plan file / explicit goal line in prompt |
| worktrees | plain `git worktree` — no managed worktree feature |
| `is_background`, other Cursor agent fields | Devin agent frontmatter: `name`, `description`, `model`, `allowed-tools` (alias `tools`), `max-nesting` |
| `SendToUser` cards, `update_state`, `api2.cursor.sh` webhooks | Devin Automation webhook triggers + org secrets; interactive prompts → `ask_user_question` |

## Model roles → Devin profiles (pstack-models.md content)

| pstack role | Devin profile | model pin |
|---|---|---|
| default worker (swarm workers, explorers that write, mechanical edits, coverage fan-out) | `pstack:worker` | `swe-2-max` (free) |
| alternate worker / cheap strong seat | `pstack:luna` | `gpt-5-6-luna-xhigh` ($0.20/M, 1M ctx; `-max` for gnarly slices) |
| trivial/bulk fan-out | `subagent_explore` or a `swe-1-7`-pinned profile | `swe-1-7` is free (262K ctx) |
| high-powered implementation (bug-fix, perf, hillclimb, precise sequences, reflect tooling) | `pstack:sol` | `gpt-5-6-sol-high` |
| judgment + prose (explainers, synthesizers, lead review, unslop-sensitive) | `subagent_general` | inherits parent |
| read-only exploration/critics when no model named | `subagent_explore` | router default (SWE-1.6) |
| panel diversity seats | `pstack:worker`, `pstack:luna`, `pstack:sol`, `pstack:terra`, `pstack:kimi`, `subagent_general` | swe-2-max / luna-xhigh / sol-high / terra-xhigh / kimi-k3-max / parent — six real families, drop seats rather than duplicating |

Devin UID spellings use dashes between version parts: `gpt-5-6-sol-high`, `gpt-5-6-luna-xhigh`, `gpt-5-6-terra-xhigh`, `swe-2-max`, `swe-1-7`, `kimi-k3-max`, `claude-fable-5-1-max`.

## Hard rules for converters

- Plugin subagents load **locally only** (CLI/Desktop, not cloud). Playbooks that fan out must say so where relevant.
- Hooks are best-effort/fail-open — nothing upstream uses them; don't add any.
- Scripts stay Bun (`#!/usr/bin/env bun`, `bun test`) — runtime is host-agnostic. Forge deps (`gh`, Graphite `gt`) stay — they're not host couplings.
- Slash-command references inside prose: `/skill` → `/pstack:skill` where it means invoking a pstack skill.
- `description` frontmatter: keep verbatim (trigger phrases still work). Only convert `disable-model-invocation` and any non-Devin fields.
- Keep prose style consistent with upstream (plain, direct, no em dashes per the 0.15.x prose pass).
