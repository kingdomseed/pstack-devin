# pstack-devin

Devin port of [pstack](https://github.com/cursor/plugins/tree/main/pstack) — rigorous agent workflows you can parallelize with confidence, fully converted to Devin primitives. Upstream: 0.15.1 (see `UPSTREAM`).

## Install

```bash
# local dev — linked to this checkout, edits apply next session
devin plugins install --local ~/repos/pstack-devin/plugins/pstack

# once pushed to a git remote
devin plugins install <owner>/<repo>#plugins/pstack
```

Skills are invoked as `/pstack:<skill>` — e.g. `/pstack:poteto-mode`, `/pstack:how`, `/pstack:deslop`.

## What's different from upstream

- Devin plugin manifest (`.devin-plugin/plugin.json`), no `.cursor-plugin/`
- Subagent profiles in `agents/` pin Devin models (`swe-2-max`, `gpt-5-6-sol-high`, …); the role→profile map lives at `~/.devin/rules/pstack-models.md` (written by `/pstack:setup-pstack`)
- Cursor Automations content → Devin Automations (app.devin.ai / v3 API)
- Bugbot triage → Devin Review (bug catcher, auto-review, autofix)
- Transcript paths → `~/.local/share/devin/cli/transcripts/` and `summaries/`
- `deslop` is bundled (upstream ships it in `cursor-team-kit`)

Plugin `agents/` profiles are local-session only (CLI + Desktop). Cloud Devin sessions get the skills and rules but not the named profiles.

## Model defaults

| role | profile | model |
|---|---|---|
| workers, fan-out, poteto-mode runner, comment review | `pstack:worker`, `pstack:poteto-agent`, `pstack:comment-sicko` | swe-2-max (free) |
| alternate worker | `pstack:luna` | gpt-5-6-luna-xhigh |
| heavy implementation | `pstack:sol` | gpt-5-6-sol-high |
| hardest tasks | `pstack:astra` | gpt-6-astra-high |
| judgment + review seat | `pstack:fable` | claude-fable-5-1-max |
| panel seats | `pstack:kimi`, `pstack:grok`, `pstack:gemini`, `pstack:terra` | kimi-k3-max, grok-4-6-xhigh, gemini-3-8-flash-high, terra-xhigh |
| judgment + prose | `subagent_general` | inherits parent |
| read-only explore | `subagent_explore` | router default |

## Refreshing from upstream

Sparse-checkout `cursor/plugins` for `pstack` (see `~/repos/cursor-plugins`), then:

```bash
rsync -a --exclude '.devin-plugin' --exclude 'AGENTS.md' \
  --exclude 'agents' --exclude 'scripts' \
  /path/to/cursor/plugins/pstack/ ./plugins/pstack/
```

Then re-apply the port to every file in `PORTING.md`'s adapted list — upstream syncs are re-port diffs by design (full conversion, no adapter layer). `PORTING.md` is the conversion contract.
