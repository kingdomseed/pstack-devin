# Running pstack on Devin

This page is the host map: how the plugin installs, what the skills are called, where model roles live, and what differs between a local session and a cloud session. Read it once, then move on to setup.

## Install

From a local checkout, install the plugin on this machine:

```bash
devin plugins install --local ./plugins/pstack
```

From a pushed repo, install it so it syncs to Devin Cloud and reaches cloud sessions:

```bash
devin plugins install owner/repo#plugins/pstack
```

The `#plugins/pstack` suffix is a `git-subdir` source: the plugin lives in a subfolder of a shared repo. Drop `--local` when the plugin should follow your personal manifest across machines and into cloud sessions.

To give one repository pstack for every Devin session that opens it, add it to that repo's `.devin/config.json` instead:

```json
{
	"requiredPlugins": ["owner/repo#plugins/pstack"]
}
```

## Names

Every pstack skill is a slash command under the plugin namespace: `/pstack:poteto-mode`, `/pstack:how`, `/pstack:interrogate`, and so on. When a guide page says `/how`, read it as `/pstack:how`.

## Model roles

`/pstack:setup-pstack` writes `~/.devin/rules/pstack-models.md`, an always-on rule that maps each role to a Devin subagent profile, one line per role. The values are profile names, not model slugs:

- `pstack:worker` (swe-2-max, free) is the default worker for mechanical edits and fan-out.
- `pstack:luna` (luna xhigh) is the cheap strong seat.
- `pstack:sol` (sol high) is the high-powered implementation seat.
- `pstack:terra` and `pstack:kimi` add two more model families for panel diversity.
- `subagent_general` inherits your parent chat model. This is the Devin spelling of "run this role on the parent". A custom profile without a `model:` field does not inherit; it falls back to the router default, so use `subagent_general` when you mean the parent.
- `subagent_explore` is the read-only profile for explorers and critics.

A panel role takes a list of profiles and runs one subagent per entry, so the list length sets the panel size. Delete a role's line to restore the shipped default.

## Local versus cloud

Plugin subagent profiles (`pstack:*`) load in local sessions only: the Devin CLI and Devin Desktop. A cloud Devin session gets the plugin's skills and rules but not its `agents/` profiles, so playbooks that fan out named profiles degrade to the built-in `subagent_general` and `subagent_explore` there.

To move work to the cloud, hand the session off:

```text
/handoff keep going until the migration check reports zero old callers
```

The cloud session picks up the conversation, branch, and uncommitted diff on its own VM, and keeps running after your laptop closes. Follow it from the terminal or at app.devin.ai.

Two more host differences worth knowing:

- Devin Review replaces bot review comments. It can auto-review a PR and autofix findings, from app.devin.ai/review or `npx devin-review <pr>`.
- Your session transcripts live at `~/.local/share/devin/cli/transcripts/` with rolling summaries under `~/.local/share/devin/cli/summaries/`. Skills that mine history, like `/pstack:recall` and `/pstack:automate-me`, read from there.

Next: [Set up pstack](./01-setup.md).
