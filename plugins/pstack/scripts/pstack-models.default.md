---
description: pstack model-role assignments — which Devin subagent profile runs each pstack role
trigger: always_on
---

# pstack model configuration. One line per role. Delete a line to fall back to the skill default.
# Values are Devin subagent profiles: pstack:worker (swe-2-max), pstack:luna (gpt-5-6-luna-xhigh),
# pstack:sol (gpt-5-6-sol-high), pstack:terra (gpt-5-6-terra-xhigh), pstack:kimi (kimi-k3-max),
# subagent_explore (read-only, router default), subagent_general (inherits the parent model).
# Profile entries in a panel list still count toward its fan-out.
feature, refactoring: pstack:worker
bug-fix: pstack:sol
perf-issue: pstack:sol
hillclimb: pstack:sol
judgment and prose: subagent_general
hardest tasks: pstack:sol
how explorer: pstack:worker
how explainer: subagent_general
how critics: pstack:luna, pstack:terra
why investigators: pstack:worker
why synthesizer: subagent_general
reflect tooling: pstack:sol
reflect judgment, divergent, synthesizer: subagent_general
arena runners: pstack:worker, pstack:luna
arena cross-judge pool: subagent_general, pstack:sol, pstack:kimi
swarm workers: pstack:worker
architect runners: pstack:sol, pstack:terra, pstack:kimi
interrogate reviewers: pstack:sol, pstack:kimi
