---
name: setup-pstack
description: Configure which models pstack uses per role. Detects your available models and writes an always-applied rule that overrides the skill defaults. Use for /setup-pstack, "configure pstack models", or changing pstack's model choices.
---

# Setup pstack

Write `~/.devin/rules/pstack-models.md`, an always-on rule that sets pstack's subagent profile per role.

pstack roles take Devin subagent profiles, not raw model IDs. The `pstack:*` profiles ship in this plugin's `agents/` directory and each pins a model. Built-in profiles `subagent_general` (inherits the parent chat model) and `subagent_explore` (read-only, router default) are always available.

## Steps

### 1. Detect available models

Run `devin models list` (`--format json` for a scriptable view). That is the dependable source for which models this account can run. A `pstack:*` profile can only serve a role when its pinned model is in the detected set. If you cannot detect any models, ask the user which models they have access to and match profiles from that. Never assign a profile whose pinned model you have not confirmed is available. `subagent_general` and `subagent_explore` always pass this check because they pin nothing.

### 2. Load current state

The default role-to-profile mapping ships in this plugin at `scripts/pstack-models.default.md`. Read it first: the file you write must keep its shape, comments, and role labels exactly. If `~/.devin/rules/pstack-models.md` already exists, read it and treat its values as the current choices. Otherwise start from the default file.

### 3. Map and confirm

Show every role with its current profile, marking any profile whose pinned model is not in the detected set as needing a choice. Ask whether to accept as-is or change specific roles, offering the usable `pstack:*` profiles plus `subagent_general` (the role runs on the parent chat model, which is how Adaptive users stay on Adaptive) and `subagent_explore` where the role is read-only. Prefer `ask_user_question` over free text.

For panel roles (how critics, arena runners, architect runners, interrogate reviewers) the value is a list, and one subagent runs per entry, so the list length sets the fan-out count. `arena cross-judge pool` is also a list, but Arena selects one value from it whose model family differs from the parent's when possible. `swarm workers` is the default profile for every worker unless a race or comparison assigns another per arm.

Never write a bare custom profile name expecting it to inherit the parent model. A custom profile without `model:` falls back to the router default (SWE-1.6), not the parent. Only `subagent_general` inherits the parent.

### 4. Validate

Every `pstack:*` value must resolve to a profile this plugin ships in `agents/`, and that profile's pinned model must be in the detected set. `subagent_general` and `subagent_explore` always pass. If a chosen profile's model is not available, stop and ask again.

### 5. Write the rule

Write `~/.devin/rules/pstack-models.md` as a copy of `scripts/pstack-models.default.md` with the confirmed substitutions applied line by line. Keep the frontmatter (`trigger: always_on`), the legend comments, and the role labels identical to the default file so the two stay consistent. Overwrite the whole file so re-runs stay idempotent. Shape:

```
---
description: pstack model-role assignments — which Devin subagent profile runs each pstack role
trigger: always_on
---

# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# <legend comments copied verbatim from scripts/pstack-models.default.md>
feature, refactoring: pstack:worker
bug-fix: pstack:sol
...
```

### 6. Offer global profiles (optional)

The plugin's `pstack:*` profiles only exist where the plugin is installed. Offer once to copy them into `~/.config/devin/agents/` so they load in every project: copy each `agents/*.md` file as-is. Ask which profiles to install rather than copying all blindly; `comment-sicko` is a single-purpose reviewer and optional. On yes, copy and confirm the paths. On no, move on.

Say this once regardless: `pstack:*` profiles load in local sessions only, the CLI and Devin Desktop. Cloud Devin sessions get this plugin's skills and rules but not its `agents/` profiles, so fan-out playbooks degrade to `subagent_general` and `subagent_explore` there.

### 7. Confirm

Tell the user the rule was written and that it applies to new sessions. Re-running this skill updates it.

### 8. Offer a verification skill (optional)

Check whether the project has a way to drive the real app for proof (a `verify-*` skill, or an existing harness). If not, offer once: "want a project-local verification skill, so agents can drive the app the way a user does and prove changes work? I can generate one with /pstack:create-verification-skill." On yes, invoke `/pstack:create-verification-skill` (resolves wherever pstack is installed: workspace, user, or plugin). On no, move on without pushing.
