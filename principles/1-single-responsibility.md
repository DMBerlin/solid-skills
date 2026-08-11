# S — Single Responsibility

> A class should have one, and only one, reason to change. — Robert C. Martin

## Translated for skills

A skill should encapsulate one capability, and one reason to change. That's
easy to say and hard to keep true once a catalog grows past a handful of
skills. Two mechanisms get you there — one is a design pattern worth
adopting, the other is a published, sourced convention:

- **A closed category taxonomy**, declared in frontmatter and validated at
  load time, so a skill can't quietly claim to be two unrelated kinds of
  thing at once. This is illustrated generically in this repo as a pattern
  worth adopting — it isn't a named external spec, and no claim here says
  otherwise.
- **A specific, single-purpose description**, so the boundary of "what this
  skill is for" is legible from one field, even without a formal taxonomy —
  this is Anthropic's actual, published convention (see below).

## What frontier labs say

Anthropic's skill-authoring guidance arrives at this from the description
side: a skill's `description` is what an agent uses to pick the right skill
"from potentially 100+ available Skills," so it has to say precisely what
the skill does and when to use it. Anthropic calls out the failure mode
directly — descriptions like `"Helps with documents"`, `"Processes data"`,
and `"Does stuff with files"` are listed explicitly as **what not to
write**, because none of them let an agent, or a human, tell where one
skill's job ends and another's begins.
[[source]](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

## Do / Don't

| Do | Don't |
|---|---|
| Give every skill either a closed, validated category, or a description specific enough that its boundary is unambiguous. | Rely on a free-text tag or a vague description nobody enforces — it drifts into a junk-drawer scope within a few skills. |
| Split a discipline ("write tests first") from its stack-specific mechanics ("JUnit + MockK") into two skills, related by extension or composition. | Bake stack-specific tooling directly into the general discipline skill "for convenience" — now every team on a different stack forks it or ignores half of it. |
| Ask, when two unrelated changes keep landing in the same skill: "does this skill have two owners now?" | Measure responsibility by file length. A short skill can still have two jobs; a long one can still have one. |

## Anti-pattern in practice

**Bad — vague scope, two jobs, no legible trigger:**

```yaml
---
name: helper
description: Helps with documents and also handles some Git stuff
---
```

Nothing here tells an agent, or a teammate, when this should fire — and it's
visibly doing two unrelated jobs (document handling, Git operations) that
will evolve on separate timelines and probably get a separate owner each.

**Good — same underlying capabilities, split and scoped:**

```yaml
---
name: processing-pdfs
description: Extracts text and tables from PDF files, fills forms, merges documents. Use when working with PDF files or when the user mentions PDFs, forms, or document extraction.
---
```

```yaml
---
name: generating-commit-messages
description: Generates descriptive commit messages by analyzing git diffs. Use when the user asks for help writing commit messages or reviewing staged changes.
---
```

A closed-taxonomy system would express the same split differently — for
example, tagging the first `kind: capability, domain: documents` and the
second `kind: workflow, domain: vcs` — but the underlying decision is
identical either way: one skill, one job, one legible trigger.

## Smell to watch for

Coupled, unrelated changes landing in the same skill on a regular cadence,
or a description you can't finish in one sentence without an "and also."
Both are the same tell: the skill's *reasons to change* have quietly
multiplied past one.
