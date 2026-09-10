---
name: setup-benny
description: Configure Benny and prepare its triage and repro automations. Use when installing Benny or changing its Slack, tracker, repository, routing, control, model, or budget settings.
triggers:
  - user
---

# Set up Benny

Benny ships as a dormant automation pack inside pstack. The plugin manifest exposes only pstack's normal skill root; this file and the two operational files are not slash skills.

The human enters setup by pointing Devin at the pack's `FOR_AGENTS.md`. The bootstrap flow copies the whole pack into the target repository, then reads this file directly at `.devin/automations/benny/skills/setup-benny/SKILL.md`.

Benny needs external configuration and two live Devin automations. Devin automations live at app.devin.ai under Settings, Automations, or in the v3 api. They are not plugin-installable.

Do not create or update an automation until the user explicitly asks. Never put a secret value in plugin files, prompts, or committed configuration. Secrets live in Devin org secrets and appear in prompts only as `${SECRET_NAME}` tokens.

## 1. Copy the pack and require shared pstack skills

Do this before asking for Benny configuration and before touching the automations editor or the v3 api.

Ask which repository will run the automations. The source pack is the directory containing `FOR_AGENTS.md`. The destination is `<target-repository>/.devin/automations/benny/`.

Merge the entire source pack into the destination:

1. Create the destination when it is absent.
2. Copy every source file to the same relative path.
3. Preserve destination-only files. Never delete unrelated files during install or refresh.
4. Keep user-owned configuration, feature maps, and routing maps outside the destination. Never overwrite them.
5. When an existing source-managed file differs, inspect the diff and merge without discarding local edits. If ownership is ambiguous, stop and ask before replacing it.
6. Verify that the destination contains `FOR_AGENTS.md`, this setup file, both operational files, their references, and the templates.

If this file is already being read from the target destination, treat the copy as complete and run the same verification before continuing.

Add pstack to the target repository's `.devin/config.json`. If the file or `.devin` directory does not exist, create it.

Merge this entry into the existing JSON or JSONC:

```json
{
	"requiredPlugins": [
		"<pstack source, for example owner/repo#plugins/pstack>"
	]
}
```

Preserve every unrelated top-level setting. If `requiredPlugins` already exists, append the pstack source without dropping other entries. Preserve comments and valid JSONC syntax when the file uses JSONC. Validate the file after editing it.

The source must resolve in Devin cloud sessions, because automation runs spawn cloud sessions, not local ones. Use a pushed GitHub repo (`owner/repo`, or `owner/repo#plugins/pstack` for a plugin in a subfolder), a git URL, or a `git-subdir` object such as `{ "source": "git-subdir", "url": "https://github.com/owner/repo.git", "path": "plugins/pstack" }`. A local path or a `devin plugins install --local` install only reaches this machine and will not load in automation sessions.

Start a fresh Devin session rooted in the target repository. Verify that these shared pstack skills resolve as `/pstack:` skills in project scope:

- `how`
- `why`
- `tdd`
- `unslop`
- `principle-separate-before-serializing-shared-state`
- `principle-minimize-reader-load`
- `principle-guard-the-context-window`
- `principle-sequence-verifiable-units`
- `principle-fix-root-causes`
- `principle-prove-it-works`

Do not count a skill loaded from the current session or a user-scoped plugin install. The check must show that a fresh session in the target repository receives pstack through the committed `.devin/config.json`.

If repo-scoped plugin requirements are unavailable or any shared dependency does not resolve, stop and explain the failure.

The Benny files are read directly from `.devin/automations/benny/`. Do not add that directory to a plugin manifest or expect its `SKILL.md` files to appear in the slash-skill list.

Tell the user that `.devin/config.json`, `.devin/automations/benny/`, and any referenced secret-free configuration must be committed before either automation is enabled. Do not commit them unless the user asks.

Once this check passes, live automation prompts may read the committed operational files by their stable repository-relative paths. They must not embed a plugin cache path or copy the file contents.

## 2. Adapt the configuration

Open these copied examples:

- `../../templates/configuration.example.yaml`
- `../reproduce-and-fix-issues/references/feature-map.example.md`

Create user-owned copies outside `.devin/automations/benny/`. These are configuration files, not pack files. Example locations:

- Project config, such as `.devin/benny/configuration.yaml`
- Project feature map, such as `.devin/benny/feature-map.md`
- Project routing map, such as `.devin/benny/routing.md`
- User config, such as `~/.config/benny/configuration.yaml`
- User feature map, such as `~/.config/benny/feature-map.md`

Fill one feature-map section for every user-facing feature the automation may reproduce. Keep it at the user point of view. Do not freeze implementation details or current code paths in the map.

Do not edit the copied examples. Pack refreshes may update source-managed files after conflict review, but they must never touch the user-owned copies.

Prefer committed, secret-free files in the target repository when a spawned automation session must read them. Otherwise paraphrase the required values into the live prompt. Reference a repository file only after you confirm that the file is committed in the repository where the automation's sessions run.

Use stable repository-relative paths for committed pack and configuration files. Never reference the plugin source directory or a plugin cache path from a live automation.

## 3. Fill the required choices

Ask for or confirm:

- Source Slack channel ID and its workspace team ID
- Optional operations or status channel ID
- Repository URL and default branch
- Triage identity or Slack user ID
- Issue tracker type, team, project, labels, and intake status
- Tracker access path, such as the org's Linear integration or a tracker MCP server
- Optional routing map path
- Required control skill name
- Required user-facing feature-map path
- Status emoji strings
- Pull request URL format
- Polling and effort budgets
- Devin session mode for triage, repro, code work, and media review

Session mode is a `devin_mode` value (`normal`, `fast`, `lite`, `ultra`, or `fusion`) applied through each automation's session settings. It is the only per-role knob an automation session gets; per-model subagent profiles do not load in cloud sessions, so do not promise a model slug per role.

The source channel, triage identity, repository, tracker access, control skill, and feature map must be explicit. Fail setup if any required value stays ambiguous.

Use pstack's `unslop` skill on the final automation names, descriptions, and prompt shims before saving them.

## 4. Check integration capabilities

The triage automation needs:

- A `slack:message` trigger on the configured source channel
- The `attach_thread` reply binding so the spawned session stays bound to the triggering thread
- The source channel in the automation's `tools.slack_channels` allowlist so the session's Slack tools can read and reply there
- Attachment access when reports include media
- Search, read, create, and update access through the configured issue tracker, such as the Linear integration (`tools.linear_enabled`) or a tracker MCP server in `tools.mcp_servers`

The repro automation needs:

- The same `slack:message` trigger and `attach_thread` binding on the source channel
- Source and operations channels in `tools.slack_channels`
- Repository read and history access through the `@{owner}/{repo}` prompt token
- A pull request path through `gh` or the org's GitHub integration, draft only
- The configured control-adapter skill

Prefer the Slack tools Devin grants through the automation's channel allowlist. The optional `BENNY_SLACK_BOT_TOKEN` may fill a narrow gap such as editing one operations status message or downloading an attachment. Store the value in Devin org secrets, not in YAML, and reference it as `${BENNY_SLACK_BOT_TOKEN}`.

Do not use undocumented integration endpoints.

## 5. Prepare the routing map

If the user wants reroutes or owner pings:

1. Copy `../triage-issue-reports/references/routing.example.md` outside `.devin/automations/benny/`.
2. Replace every placeholder with public or organization-local values.
3. Keep owner pings off by default.
4. Allow a ping only for a configured feature owner or a confirmed likely regression author.

If no routing map is configured, triage may classify a report but must not guess a destination or owner.

## 6. Verify the control adapter

Read `../reproduce-and-fix-issues/references/control-adapter.md` and the user's completed feature map.

Confirm that the named skill can:

- Bring up the target app
- Navigate every mapped feature through the real UI
- Exercise mapped states through declared adapter actions
- Inspect state without forcing the result
- Capture screenshots
- Start and stop a recording
- Clean up its processes and temporary data

If any capability is missing, leave the repro automation disabled. It must fail closed rather than claim a reproduction it did not perform.

## 7. Prepare the live automations

Ask whether this is first-time creation or configuration of existing automations.

Read `../../FOR_AGENTS.md` from the copied pack as the primary user-intent source for either path. Use it to understand the two triggers, tools, instructions, outcomes, and shared rules.

### First-time creation

Create one automation at a time, either in the app.devin.ai automations editor (Settings, Automations) or through `POST /v3/organizations/{org}/automations` with a service-user key that has `ManageOrgAutomations`.

For each automation:

1. Read the matching copied prompt template as secondary internal source material.
2. Turn `FOR_AGENTS.md`, the finished Benny configuration, and the template intent into a complete prompt for the `start_session` action.
3. Tell the live prompt to read and follow its exact committed operational file under `.devin/automations/benny/`, referenced with an `@{owner}/{repo}` repo token so the session clones the right repository.
4. Use the stable repository-relative path, not a plugin source or cache path. Do not copy the operational file contents into the live prompt.
5. Confirm that the copied pack and any referenced configuration files are committed in the same repository the prompt token names.
6. Draft the full trigger, condition, reply-binding, action, tools, and limits fields. Show the user the complete draft before saving it.
7. Save the automation disabled, or with `enabled: false` through the api. Enable it only after the thread-safety test in section 8 passes.
8. Finish and review this automation before starting the next one.

A `slack:message` trigger fires on every message in scope, so narrow it with `conditions` to the configured source channel and, when the schema exposes it, to top-level messages. Discover the exact condition field names and reply verbs for `slack:message` through `GET /v3/organizations/{org}/automations/schemas` rather than guessing.

The triage draft, filled from configuration:

- Name `benny-triage`.
- Trigger `slack:message` conditioned to the configured source channel.
- Reply binding `attach_thread` on the trigger.
- Action `start_session` whose prompt reads and follows `.devin/automations/benny/skills/triage-issue-reports/SKILL.md` for every run.
- `tools.slack_channels` limited to the source channel.
- Tracker access through `tools.linear_enabled` or a tracker MCP server.
- The prompt instructs the session to read the triggering thread, reply only inside it, classify, inspect evidence, trace cause, dedupe, and create only clear new bugs.
- One thread-only verdict ending with the configured `[benny:bug]`, `[benny:performance]`, or `[benny:other]` marker and optional tracker URL.
- Never post a source-channel root message.

After the triage automation is saved, the repro draft:

- Name `benny-reproduce`.
- The same `slack:message` trigger on the configured source channel, with `attach_thread`.
- Action `start_session` whose prompt reads and follows `.devin/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` for every run.
- `tools.slack_channels` covering the source channel and the optional operations channel.
- The configured repository and default branch through the `@{owner}/{repo}` prompt token.
- Tracker, control-adapter, and feature-map requirements paraphrased into the prompt unless an eligible committed file exists in the same repository.
- Wait for a trusted triage marker before acting.
- Reproduce the exact symptom twice through the mapped real UI and capture evidence.
- Verify an existing fix without authoring over it.
- Attempt an optional bounded fix only after confirmed repro, then open a draft pull request when proof and checks pass.
- Never post a source-channel root message.

Example api body for the triage automation:

```json
{
	"name": "benny-triage",
	"enabled": false,
	"run_as": { "type": "organization" },
	"triggers": [
		{
			"event_type": "slack:message",
			"conditions": {
				"any": [
					{ "all": [
						{ "field": "channel_id", "operator": "eq", "value": "C0SOURCE" }
					] }
				]
			},
			"replies": [ { "type": "attach_thread" } ]
		}
	],
	"actions": [
		{
			"type": "start_session",
			"prompt": "In @{owner}/{repo}, read and follow the committed file .devin/automations/benny/skills/triage-issue-reports/SKILL.md for this run. The triggering Slack event payload is attached. <paraphrased configuration and intent>",
			"session": { "tags": ["benny", "triage"] }
		}
	],
	"tools": {
		"slack_channels": [ { "team_id": "T0TEAM", "channel_id": "C0SOURCE" } ],
		"linear_enabled": true
	},
	"concurrency": { "max_concurrent_runs": 4 },
	"limits": { "invocations": { "max_per_window": 20, "window_seconds": 3600 } },
	"notifications": { "email": { "when": "dispatch_failed", "recipients": ["you@example.com"] } },
	"session_settings": { "devin_mode": "normal" }
}
```

```bash
curl -X POST "https://api.devin.ai/v3/organizations/$ORG_ID/automations" \
	-H "Authorization: Bearer $DEVIN_API_KEY" \
	-H "Content-Type: application/json" \
	-d @benny-triage.json
```

Condition field names such as `channel_id` come from the event schemas endpoint; confirm them there before saving.

### Existing automations

There is no draft-review skill for updates. Validate the configuration, routing, control-adapter, and feature map first, then give the user this concise editor checklist. The same fields map to `PATCH /v3/organizations/{org}/automations/{automation_id}` for api-driven updates.

For the existing triage automation, update:

- Name and description
- Prompt: direct instruction to read `.devin/automations/benny/skills/triage-issue-reports/SKILL.md`
- `slack:message` trigger conditioned to the source channel, with `attach_thread`
- `tools.slack_channels` limited to the source channel
- Issue-tracker access through `tools.linear_enabled` or a tracker MCP server
- Paraphrased triage instructions, thread-only rule, and Benny verdict markers

For the existing repro automation, update:

- Name and description
- Prompt: direct instruction to read `.devin/automations/benny/skills/reproduce-and-fix-issues/SKILL.md`
- Matching `slack:message` trigger and source channel, with `attach_thread`
- Repository and default branch through the `@{owner}/{repo}` token
- `tools.slack_channels` covering source and operations channels
- Draft pull request capability
- Tracker, control-adapter, and feature-map requirements
- Paraphrased marker wait, evidence, verification, and bounded-fix instructions

Ask the user to update each existing automation directly in its automations editor or through the api. Do not create replacements or duplicates.

### Creation boundary

Never call an undocumented backend endpoint. Never hand-edit an automation's stored definition outside the editor or the v3 api. For new automations, the only finish paths are the app.devin.ai automations editor and `POST /v3/organizations/{org}/automations`.

Do not enable either automation until the thread-safety test passes after the save.

## 8. Test thread safety

Use a test channel or a harmless test report.

Before testing, confirm that the target repository's `.devin/config.json`, `.devin/automations/benny/`, and every referenced secret-free configuration file are committed on the branch the automation's sessions will clone. Confirm that both live prompts point at their exact committed operational files. If any check fails, stop. Tell the user that the automation cannot be enabled yet.

Verify:

1. Triage reads the triggering thread's coordinates from the event payload and posts exactly one verdict as a reply.
2. The verdict contains one configured marker.
3. Repro accepts the marker only from the configured triage identity.
4. Repro keeps the same immutable source coordinates.
5. No source-channel root message appears.
6. A delegated subagent cannot use any Slack write action.
7. Missing coordinates, a deleted parent, or a failed preflight produces no post and no tracker issue.

Enable normal traffic only after all seven checks pass.

PORT-NOTE: the `monitor_session` (Triage Devin) action is a managed alternative to the triage `start_session` design, but it owns its own reply style and does not follow this pack's marker contract, so Benny keeps `start_session`.
