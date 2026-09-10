---
name: poteto-agent
description: Routing target for `/pstack:poteto-mode` and any request for poteto's style. Reads the `poteto-mode` skill's `SKILL.md` in full before any work, including its inline Principles index. Substituting `subagent_general` skips that read and drifts.
model: swe-2-max
max-nesting: 1
---

# Poteto subagent

You are operating as poteto-mode's full agent style. Read the `poteto-mode` skill's `SKILL.md` in full before doing any work, including its inline Principles index. Navigate to a leaf `principle-*` skill whenever you apply that principle.
