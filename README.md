# SOLID Skills

Design principles for the unit of composition inside an agentic system — the
**skill**, not the whole agent.

[12-Factor Agents](https://github.com/humanlayer/12-factor-agents) is about
how an *agent* should be built: own your prompts, own your context window, own
your control flow. Its factor 10, "Small, Focused Agents," is essentially
Single Responsibility applied to the whole agent.

This repo is about the layer most agent frameworks put underneath that: the
individually invokable **skill** — a discrete capability that gets selected,
loaded into context, and composed with other skills to build up a task. That
composition layer is exactly the kind of problem object-oriented design spent
decades thinking about. So: what happens if you take the five SOLID
principles and ask what they mean for a skill?

Some of it is a genuinely tight fit — closer to a literal description of good
architecture than a metaphor. Some of it stretches. Each principle below says
which, with concrete do/don't examples.

## The five

| # | Principle | One-liner for skills |
|---|-----------|----------------------|
| S | [Single Responsibility](principles/1-single-responsibility.md) | One skill, one reason to change — enforced by a closed category taxonomy, not a style guideline. |
| O | [Open/Closed](principles/2-open-closed.md) | New capabilities extend the catalog by being added and tagged, never by editing what already selects them. |
| L | [Liskov Substitution](principles/3-liskov-substitution.md) | A specialized skill should honor its base's contract — provable only where you actually test for structural equivalence, aspirational everywhere else. |
| I | [Interface Segregation](principles/4-interface-segregation.md) | A skill's always-loaded core should be thin; deeper capability loads on demand, not by default. |
| D | [Dependency Inversion](principles/5-dependency-inversion.md) | Skills depend on abstract capability names, not on each other's file paths; the system that selects skills should depend on data shapes, not on disk. |

## Where this bends

SOLID was built for statically typed, compiled, polymorphic code. A skill is
a natural-language artifact, interpreted probabilistically by a model — not
compiled and type-checked. Four of these five principles have real,
buildable, often test-enforceable counterparts once you're managing a catalog
of skills rather than one monolithic prompt. Liskov Substitution is the
honest exception: a useful frame for what you want, with no general
mechanism that proves you have it, only specific slices you can choose to
test for. Say clearly which ones hold. Don't force the ones that don't.

## Using this repo

Each principle page has the same shape:

- the classic definition
- what it means once "class" becomes "skill"
- a do/don't table
- a worked example
- a smell to watch for

None of the examples reference a specific product or company — they're
deliberately generic so the patterns transfer.

## Related reading

- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents) — the
  sibling document, one layer up (the agent, not the skill).
