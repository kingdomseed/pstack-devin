# pstack

When the user says poteto-mode, $poteto-mode, /poteto-mode, or pstack, read and follow the pstack plugin's `poteto-mode` skill (`/pstack:poteto-mode`), then keep applying it for the rest of the session unless they opt out.

Model roles live in `~/.devin/rules/pstack-models.md`. One line per role; the value is a Devin subagent profile or `subagent_general` to inherit the parent model. Default workers: `pstack:worker` (swe-2-max, free). High-powered implementation: `pstack:sol` (gpt-5-6-sol-high). Judgment and prose stay on the parent via `subagent_general`.

Plugin subagent profiles (`pstack:*`) load in local sessions only — the CLI and Devin Desktop. Cloud Devin sessions get this plugin's skills and rules but not its `agents/` profiles; fan-out playbooks degrade to built-in profiles there.

Before every commit, run the pstack `deslop` skill (`/pstack:deslop`). It is bundled. Do not skip it.
