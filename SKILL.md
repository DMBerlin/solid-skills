---
name: solid-skills
description: Audits and refactors agent skill definitions (SKILL.md files, tool and function definitions, prompt-based capabilities) against five SOLID-derived design principles for maintainability and extensibility. Use when asked to review, refactor, clean up, restructure, or improve the design of a skill catalog, agent tool definitions, or prompt library, or to bring an ad-hoc set of skills up to a production standard.
---

# Refactoring skills against SOLID

This skill turns an ad-hoc set of agent skills, tools, or prompt-based
capabilities into a maintainable, extensible catalog. It applies five
principles borrowed from object-oriented design and adapted for the skill
layer of an agentic system. Each principle has its own reference page with
the classic definition, sourced frontier-lab guidance, a do-and-don't
table, and a worked anti-pattern. Load the relevant page before judging a
skill against that principle. Don't rely on the one-line summaries below as
the whole standard.

- [Single Responsibility](principles/1-single-responsibility.md): one skill, one reason to change.
- [Open/Closed](principles/2-open-closed.md): new capabilities extend the catalog without editing what already selects or lists them.
- [Liskov Substitution](principles/3-liskov-substitution.md): a specialized skill should honor its base's contract, provably only where you test for it.
- [Interface Segregation](principles/4-interface-segregation.md): the always-loaded surface stays thin; deeper capability loads on demand.
- [Dependency Inversion](principles/5-dependency-inversion.md): skills depend on abstract capabilities, not hardcoded paths or assumed environments.

## Procedure

Given a set of skill, tool, or prompt files to audit:

1. Inventory every file. For each one, write its stated purpose in one
   sentence, in your own words, not by quoting its description back. If
   you can't state the purpose in one sentence, that's already a Single
   Responsibility signal.

2. Check each file against each principle, one at a time, loading that
   principle's page first.
   - Single Responsibility: does the file do one job with one legible
     trigger, or does its description need an "and also"?
   - Open/Closed: does adding a sibling capability require editing this
     file, or a manifest that lists it, or does it just get added and
     discovered?
   - Liskov Substitution: if this file extends or specializes another, is
     there a shared, non-negotiable contract, and is it enforced anywhere,
     or only assumed?
   - Interface Segregation: does the always-loaded content stay to the
     common path, with rare or advanced material split into files loaded
     on demand, or does everything load every time?
   - Dependency Inversion: does the file depend on an abstract capability
     name resolved elsewhere, or does it hardcode a specific tool, path, or
     assume something about the environment it happens to run in?

3. For each violation, propose a specific fix, citing the principle and
   pointing at the matching anti-pattern in that principle's page. Don't
   propose a generic "improve this" note. Name the split, the namespace,
   the extracted contract, the file boundary, or the abstraction that fixes
   it.

4. Re-check every proposed fix against the other four principles before
   finalizing it. A fix that satisfies one principle and breaks another
   (for example, splitting one overloaded skill into ten that all duplicate
   the same untested contract) is not a fix.

5. Report findings as a table: skill, principle violated, the smell
   observed, the proposed fix, and a one-line reason. Group by skill, not
   by principle, so a reader can see everything wrong with one file at
   once.

## Scope and honesty

Apply principles in proportion to the catalog's actual size and risk. A
three-skill personal project does not need a closed category taxonomy; a
fifty-skill shared catalog probably does. Say when a principle doesn't
clearly apply rather than forcing it.

Match whatever convention the target system already uses (a taxonomy, a
bare description field, a folder-per-domain layout) instead of importing a
taxonomy from this repo's own examples. The principle is the point, not the
specific spec that illustrates it here.

Liskov Substitution claims need a test to back them. If you recommend
extracting a shared contract, say so only where you can also specify what a
build check for it would verify. An untested "these should behave the same"
is not a fix. It's a hope.

Don't claim a refactor is complete without reading the actual file content
given. Zero fabrication: don't invent a taxonomy, a missing capability, or
a project convention that wasn't in what you were handed.

## Using this outside a skill-aware runtime

If a skill loader picked this file up automatically, load the linked
principle pages as you reach each check in the procedure above.

If someone pasted this repository, or a link to it, into a chat instead,
apply the same procedure by hand: read the principle pages as reference
before judging a skill against them, treat the one-line summaries above as
an index and not the full standard, and produce the same table-format
report.
