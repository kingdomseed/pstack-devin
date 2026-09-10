# benny

benny gives you two devin automations for slack issue reports. one triages each report. the other reproduces confirmed bugs and may prepare a small draft fix.

the files in this directory are dormant setup and automation sources. they do not appear as slash skills.

devin automations live at app.devin.ai under Settings, Automations, or in the v3 api. they are not plugin-installable; this pack ships the setup instructions and prompt templates instead.

## set it up

1. point devin at [`FOR_AGENTS.md`](./FOR_AGENTS.md) and name the target repository.
2. let setup merge this whole directory into the target at `.devin/automations/benny/`. it must preserve destination-only files and review conflicts instead of overwriting local edits.
3. let setup add pstack to the target repository's `.devin/config.json` so devin sessions there load the shared skills:

```json
{
	"requiredPlugins": [
		"<pstack source, for example owner/repo#plugins/pstack>"
	]
}
```

the entry must resolve in cloud sessions, so use a pushed repo source, not a `--local` install.

4. keep user-owned configuration outside the copied pack, for example in `.devin/benny/`. adapt [`configuration.example.yaml`](./templates/configuration.example.yaml) and [`feature-map.example.md`](./skills/reproduce-and-fix-issues/references/feature-map.example.md). keep secrets in devin org secrets and reference them as `${SECRET_NAME}` tokens.
5. commit `.devin/config.json`, `.devin/automations/benny/`, and any secret-free configuration before enabling either automation.
6. create each automation in the app.devin.ai automations editor or through `POST /v3/organizations/{org}/automations`, then send a harmless test report and verify every source-channel post stays in the original thread.
