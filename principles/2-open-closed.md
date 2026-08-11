# O — Open/Closed

> Software entities should be open for extension, but closed for
> modification. — Bertrand Meyer (via Robert C. Martin)

## Translated for skills

If the thing that decides "which skills does this agent have access to" is a
**query or a directory scan** — pick up every skill dropped into a folder,
or every skill matching a tag — rather than a **hand-maintained list of
skill names baked into configuration**, then adding a new skill makes it
available immediately, with zero edits to anything that already exists. The
registry is closed: you never modify it to onboard a new capability. The
catalog is open: drop in a new, correctly described skill and it's
reachable.

The other half is specialization by extension or composition: a base skill
states a general contract; a specialized skill builds on it — through an
explicit "extends" relationship, or simply by depending on it — without the
base ever needing to know the specialization exists.

## What frontier labs say

Anthropic's Skills architecture is built around exactly this: skills are
discovered by scanning a directory and pre-loading each one's metadata into
the system prompt — the runtime never needs a hand-written manifest naming
every skill in advance. Anthropic's tool-naming guidance pushes the same
idea from a different angle: namespace tools by service or resource
(`asana_search`, `jira_search`, `asana_projects_search`) precisely so that
adding a new tool for a new service never requires renaming or restructuring
the tools that already exist.
[[source]](https://www.anthropic.com/engineering/writing-tools-for-agents)

## Do / Don't

| Do | Don't |
|---|---|
| Discover skills by directory scan or metadata query at load/startup time. | Maintain a static, per-project list of skill names that has to be hand-edited every time a new skill ships. |
| Namespace related tools/skills by service or domain (`asana_search`, `jira_search`) so a new integration adds a new namespace, not a redesign of an existing one. | Give every tool a flat, generic name (`search`, `search2`) that collides the moment a second integration needs the same verb. |
| Let a specialized skill declare what it extends or depends on; keep the base ignorant of every specialization that exists. | Modify a base skill's content every time a new variant is needed — that's a modification, not an extension. |

## Anti-pattern in practice

**Bad — a second integration forces a rename of the first:**

```
tools:
  search        # originally: Asana search
  search_2      # now: Jira search, bolted on
  search_v2_new # now: Linear search, whoever's turn it was to be creative
```

Every new integration is a naming collision to be worked around, not a
clean addition.

**Good — namespaced from the start:**

```
tools:
  asana_search
  jira_search
  linear_search
```

Adding `notion_search` next month touches nothing that already exists.

## Smell to watch for

A "supported skills/tools" list, or a naming scheme, that needs an edit or a
rename every time someone adds a capability. If onboarding a new one
requires touching more than the new thing itself, the system isn't
open/closed — it's just a list.
