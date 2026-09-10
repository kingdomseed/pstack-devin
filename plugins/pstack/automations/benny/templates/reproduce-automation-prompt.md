# Reproduce automation prompt

> Source material for the copied setup workflow. Paraphrase this intent into the `start_session` prompt of a Devin automation, in the app.devin.ai automations editor or a `POST /v3/organizations/{org}/automations` body, after you confirm that the copied pack is committed in the repository where the automation's sessions will run.

Read and follow `.devin/automations/benny/skills/reproduce-and-fix-issues/SKILL.md` in @{owner}/{repo} for this run.

Configuration source. Include this repository-relative path only when it is committed in the same target repository. Otherwise paraphrase the configured values. Never use a plugin source or cache path:

```text
{{BENNY_CONFIG_PATH}}
```

Trigger: `slack:message` conditioned to the configured source channel, with the `attach_thread` reply binding. The triggering event's payload is appended to the session's prompt automatically and carries the channel and thread coordinates:

```json
{
	"source_channel_id": "{{SLACK_CHANNEL_ID}}",
	"message_ts": "{{SLACK_MESSAGE_TS}}",
	"thread_ts": "{{SLACK_THREAD_TS_OR_EMPTY}}"
}
```

The prompt should describe this as a new top-level report in the configured source Slack channel. It should include the configured repository, default branch, issue tracker, control adapter, feature map, and draft pull request capability.

Treat the source channel and root thread timestamp as immutable. If either is missing or does not match configuration, stop without posting.

Wait for a configured triage marker from the configured triage identity in this exact thread. Proceed only for `[benny:bug]` or `[benny:performance]`.

Require the configured control-adapter skill before attempting a repro. Reproduce the exact discriminating symptom twice through the real UI. Verify existing pull requests or commits without authoring over them. Attempt a bounded fix only after a confirmed repro and the operational file's fix gate.

The coordinator is the only Slack poster. Every child prompt must forbid `chat.postMessage` and all other Slack write actions. Children return findings only.

Never post a root message in the source channel.
