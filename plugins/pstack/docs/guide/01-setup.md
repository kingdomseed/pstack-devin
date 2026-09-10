# Set up pstack

In this page you install the plugin, pick which models pstack uses, and run your first task. Setup is one command plus a short conversation.

## Install the plugin

From a local checkout:

```bash
devin plugins install --local ./plugins/pstack
```

Or from a pushed repo, as a git-subdir source:

```bash
devin plugins install owner/repo#plugins/pstack
```

Devin shows what the plugin adds and confirms the install. Installed plugins load at the start of the next session. [Running pstack on Devin](./00-devin.md) covers install scopes and the local-versus-cloud differences.

## Pick your models

Run:

```text
/pstack:setup-pstack
```

[`/pstack:setup-pstack`](../../skills/setup-pstack/SKILL.md) detects the models you have access to, shows you each role (code delegates, judgment, the review panels), and asks what you want. Answer the questions. It writes `~/.devin/rules/pstack-models.md`, a small always-on rule every pstack skill reads.

You only override what you care about. A role with no line in the rule keeps the skill's default. To restore a default later, delete that role's line, or just run `/pstack:setup-pstack` again.

Each role's value is a Devin subagent profile, not a model slug. Set a role to `subagent_general` and it runs on your parent chat model; `inherit-parent` and `auto` are accepted aliases for the same thing. For a panel role the value is a list of profiles, and one subagent runs per entry, so the list length sets the panel size. Setup also configures `swarm workers`, the default profile for every `/pstack:swarm` worker unless a race names a profile for each arm.

## Accept the verification offer, or don't

At the end of setup, `/pstack:setup-pstack` looks for a way to prove app behavior in your project, either a `verify-*` skill or an existing harness. If it finds neither, it offers once to generate one with [`/pstack:create-verification-skill`](../../skills/create-verification-skill/SKILL.md).

Say yes and it writes `.devin/skills/verify-<app>/`, a project-local skill that teaches agents to drive your app the way a user does. It proves the skill works once before handing it over. Say no and setup moves on. You can run `/pstack:create-verification-skill` yourself any time. [Verify and ship](./06-verify-and-ship.md#create-a-project-verification-skill) covers when it earns its place.

After setup, start a new session. The model rule applies to new sessions.

## Run your first task

Pick something real but small, and describe it the way you'd describe it to a colleague:

```text
/pstack:poteto-mode add a --json flag to this command. text output stays byte-identical. verify both.
```

Watch the todo list. Its first items are the matched playbook's steps copied in, the Feature playbook for this prompt. If `/pstack:poteto-mode` skips a step, the step stays in the list with `skip: <reason>`, so you can see what it chose not to do.

From here you can type normal follow-ups. `/pstack:poteto-mode` is sticky. It stays on for the conversation until you opt out by saying so.

Next: [Route work through `/pstack:poteto-mode`](./02-poteto-mode.md).
