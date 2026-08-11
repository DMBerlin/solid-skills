# O — Open/Closed

> Software entities should be open for extension, but closed for
> modification. — Bertrand Meyer (via Robert C. Martin)

## Translated for skills

If the thing that decides "which skills does this project need" is a
**query over metadata** ("every discipline skill, plus everything tagged for
this stack") rather than a **hand-maintained list of skill names**, then
adding a new skill in an existing category makes it available everywhere
that category is queried — with zero edits to any existing configuration.
The configuration is closed: you never modify it to onboard a new
capability. The catalog is open: a new, correctly tagged skill is reachable
the moment it exists.

The other half of this principle is specialization by extension: a base
skill states a general contract; a specialized skill builds on it without
the base ever needing to know the specialization exists.

## Do / Don't

| Do | Don't |
|---|---|
| Select skills by querying metadata (category, stack, always-on scope) at resolution time. | Maintain a static, per-project list of skill names that has to be hand-edited every time a new skill ships. |
| Let a specialized skill declare what it extends, and let the base stay ignorant of every specialization that exists. | Modify a base skill's content every time a new stack needs a variant — that's a modification, not an extension. |
| Treat "add a new tagged skill" as a change with a blast radius of one file. | Treat "add a new capability" as a change that requires touching N existing configs to wire it up. |

## Worked example

Adding a Kafka-flavored TDD specialization:

```yaml
---
name: test-driven-development-kafka
kind: stack
stack: backend-kafka
extends: test-driven-development
scope: contextual
---
# Embedded-broker test conventions for event-driven consumers/producers.
```

Nothing about `test-driven-development` changes. Nothing about any existing
project configuration that already selects `kind: discipline` skills
changes. The new skill exists, is tagged, and becomes selectable the moment
it's added — that's the whole diff.

## Smell to watch for

A "supported skills" list that needs an edit every time someone adds a
capability. If onboarding a new skill requires touching more than the skill
itself, the catalog isn't open/closed — it's just a list.
