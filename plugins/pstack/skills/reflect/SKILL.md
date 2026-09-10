---
name: reflect
description: Spawn three parallel review subagents over the active transcript, surface learnings, and route each to a concrete edit on an existing skill. Use when the user says reflect.
triggers: [user]
---

# Reflect

Mine the current conversation for durable learnings, then route them into skill edits.

## When to invoke

Invoke when the user says "reflect" or "/reflect". Skip when the conversation is trivial, off-topic, or already covered by an existing skill the parent followed correctly. One-offs are not learnings.

## Process

### 1. Locate the active transcript

The parent finds its own transcript file before fanning out. Devin CLI transcripts live in `~/.local/share/devin/cli/transcripts/*.json`; session summaries live in `~/.local/share/devin/cli/summaries/history_*.md`. Use the current session's files. Do not trawl the whole directory for other sessions' transcripts. That crosses session boundaries and reads private chats from unrelated work.

```bash
ls -t ~/.local/share/devin/cli/transcripts/*.json ~/.local/share/devin/cli/summaries/history_*.md 2>/dev/null | head -10
```

For each candidate, check that its opening user message matches this conversation's first user prompt. Take the matching path. If no path resolves, write a tight digest of the session and pass that instead.

### 2. Spawn three reviewers in parallel

One message, three `run_subagent` calls, `is_background: true` on each. Reviewers need MCP access for context lookups (tickets, chat threads, observability traces referenced in the transcript), so the profile must carry MCP tools. `subagent_explore` strips them; a profile whose `allowed-tools` omits MCP calls does the same. `subagent_general` always has full access.

| Lens | `profile` | Prompt template |
|---|---|---|
| Judgment | your configured `reflect judgment` entry (default `subagent_general`) | `references/judgment-reviewer.md` |
| Tooling | your configured `reflect tooling` entry (default `pstack:sol`) | `references/tooling-reviewer.md` |
| Divergent | your configured `reflect judgment` entry (default `subagent_general`) | `references/divergent-reviewer.md` |

Configured entries come from `~/.devin/rules/pstack-models.md`. Pass each template verbatim, substituting the transcript path or digest where marked. Reviewers return findings in the `run_subagent` result.

### 3. Synthesize

One `run_subagent` call on your configured `reflect judgment` entry (default `subagent_general`). The synthesizer's quality check includes spot-verifying citations, which can require MCP access, so the same MCP-capable-profile rule applies. Use `references/synthesizer.md` verbatim, with each reviewer's full output inlined where marked. The synthesizer returns a structured Accepted / Rejected / Backlog list.

### 4. Structural enforcement check

Sanity-check the synthesizer's Accepted list. For any item that would be enforced more reliably by a lint rule, script, metadata flag, or runtime check, move it from Accepted to Backlog. See the **principle-encode-lessons-in-structure** skill.

### 5. Apply

Before applying any Accepted edit, present the synthesizer's full Accepted/Rejected/Backlog output to the user and wait for explicit approval. The user picks which subset to apply and may redirect routings. Skill changes affect every future agent in the org. Do not auto-apply.

Backlog items file to whatever devex / backlog tracker your team uses automatically. Only the Accepted list waits for approval.

For each approved Accepted item, follow the Routing field exactly:

- Trivial existing-skill edit (a one-line bullet, a tightened sentence, a stale fact corrected): parent does directly.
- Substantive existing-skill edit (a new section, a new pattern table, more than ~10 lines): hand to a `run_subagent` worker on `subagent_general` with the skill path, the proposed change, and Devin's SKILL.md format (frontmatter `name`, `description`, `triggers`, `model`, `allowed-tools`), then review the diff before keeping it.
- `tune description: <skill path>` (the skill exists but didn't trigger when it should have): same hand-off, scoped to the `description` field. Draft the new description, check it against the trigger phrases that should have fired, iterate.
- `new skill: <kebab-name>`: author the SKILL.md directly under the target skills directory (`.devin/skills/` for project, `~/.config/devin/skills/` for global), in Devin's format. Do not invent the shape ad hoc.

If your environment ships a SKILL.md validator, run it on every touched skill before declaring done. Skip this step if it doesn't.

### 6. Summarize for the user

Short list, no preamble:

- Edits applied: `<skill path>`. What changed, one line each.
- New skills created: `<skill path>`. One line each (rare).
- Backlog filed to the devex tracker: `<issue title>` (`<tags>`). One line each.
- Dropped: one line per rejected finding + reason from the synthesizer.

PORT-NOTE: Devin ships no bundled create-skill agent, so substantive edits and new skills are authored directly (or delegated via `run_subagent`) using Devin's SKILL.md frontmatter format.
