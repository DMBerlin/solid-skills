# S — Single Responsibility

> A class should have one, and only one, reason to change. — Robert C. Martin

## Translated for skills

A skill should encapsulate one capability, and one reason to change. That's
easy to say and hard to keep true once a catalog grows past a handful of
skills — the useful move isn't the platitude, it's giving every skill a
**closed, mutually exclusive category** it must declare before it's even
valid: is this a workflow trigger, a cross-cutting discipline, a
language/stack specialization, or a domain fact? A category that isn't
closed (a free-text field, tags only) lets a skill quietly drift into a
second job with nothing in the system able to say "no."

## Do / Don't

| Do | Don't |
|---|---|
| Give every skill exactly one category from a closed, validated set, and reject a skill at load time if it's missing or invalid. | Let a skill's category be free-text that nobody enforces — it drifts into a junk-drawer tag within a few skills. |
| Split a discipline ("write tests first") from its stack-specific mechanics ("JUnit + MockK") into two skills related by extension. | Bake stack-specific tooling directly into the discipline skill "for convenience" — now every team on a different stack forks it or ignores half of it. |
| Ask, when two unrelated changes keep landing in the same skill: "does this skill have two owners now?" | Measure responsibility by file length. A short skill can still have two jobs; a long one can still have one. |

## Worked example

```yaml
---
name: test-driven-development
kind: discipline
scope: contextual
---
# Practice test-first development. Stack-agnostic philosophy only.
```

```yaml
---
name: test-driven-development-jvm
kind: stack
stack: backend-jvm
extends: test-driven-development
scope: contextual
---
# JUnit 5 + MockK conventions. Defers to the base discipline for philosophy.
```

Two files, two owners, two reasons to change. A change to "how we feel about
mocking" touches the discipline skill. A change to "which JVM test runner we
standardize on" touches the stack skill. Neither PR touches the other file.

## Smell to watch for

Coupled, unrelated changes landing in the same skill on a regular cadence.
That's the tell that a skill's *reasons to change* have quietly multiplied
past one, category tag or not.
