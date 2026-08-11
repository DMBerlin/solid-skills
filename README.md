# SOLID Skills

Design principles for the unit of composition inside an agentic system — the
**skill**, not the whole agent, and not any one vendor's implementation of
it.

[12-Factor Agents](https://github.com/humanlayer/12-factor-agents) is about
how an *agent* should be built: own your prompts, own your context window,
own your control flow. Its factor 10, "Small, Focused Agents," is
essentially Single Responsibility applied to the whole agent.

This repo is about the layer most agent frameworks put underneath that: the
individually invokable **skill** — a discrete capability that gets
discovered, loaded into context, and composed with other skills to build up
a task. That composition layer is exactly the kind of problem object-oriented
design spent decades thinking about. So: what happens if you take the five
SOLID principles and ask what they mean for a skill?

Some of it is a genuinely tight fit — closer to a literal description of
good architecture than a metaphor. Some of it stretches. Each principle page
says which, backs it with **do *and* don't** examples, and grounds it in
what frontier labs have actually published about writing good skills and
tools — not just frontmatter shape, but description quality, body content,
and file structure.

## Not tied to one framework

Different skill systems express the same underlying ideas with different
mechanics — a closed category taxonomy in one, a free-text `description`
field in another, a folder-per-domain convention in a third. Every principle
page below illustrates the idea with **more than one real convention** —
including Anthropic's own published Agent Skills spec (`name` +
`description` frontmatter, gerund naming, progressive disclosure) alongside
a generic metadata-taxonomy style — so the takeaway is the underlying
property, not the syntax that happens to express it in any one tool.

## The five

| # | Principle | One-liner for skills |
|---|---|---|
| S | [Single Responsibility](principles/1-single-responsibility.md) | One skill, one reason to change, one legible trigger — not three jobs wearing one description. |
| O | [Open/Closed](principles/2-open-closed.md) | New capabilities extend the catalog by being added, tagged, or dropped in a folder — never by editing what already selects or lists them. |
| L | [Liskov Substitution](principles/3-liskov-substitution.md) | A specialized skill should honor its base's contract — provable only where you actually test for structural equivalence, aspirational everywhere else. |
| I | [Interface Segregation](principles/4-interface-segregation.md) | A skill's always-loaded surface should be thin; deeper capability loads on demand, not by default. |
| D | [Dependency Inversion](principles/5-dependency-inversion.md) | Skills depend on named abstractions — capabilities, other tools by qualified name — never on a hardcoded path or an assumed environment. |

## Where this bends

SOLID was built for statically typed, compiled, polymorphic code. A skill is
a natural-language artifact, interpreted probabilistically by a model — not
compiled and type-checked. Four of these five principles have real,
buildable, often test-or-eval-enforceable counterparts once you're managing
a catalog of skills rather than one monolithic prompt — and, encouragingly,
frontier labs' own published skill-authoring guidance independently
converges on several of them (progressive disclosure, unambiguous
descriptions, avoiding overlapping capabilities) without ever mentioning
SOLID by name. Liskov Substitution is the honest exception: none of the
sources below describe anything resembling a general substitutability
guarantee across skill specializations. That absence is itself evidence for
treating it as the principle that stretches furthest.

## Using this repo

Each principle page has the same shape:

- the classic definition
- what it means once "class" becomes "skill"
- what frontier labs have independently published about it
- a do/don't table
- worked examples, including an explicit anti-pattern
- a smell to watch for

Examples name real, widely known products (Jira, Asana, BigQuery, JUnit,
GitHub) only as familiar reference points for illustrating a pattern — none
of them tie a principle to one proprietary framework, internal system, or
company's specific architecture. The principle is the point; the product
names are just easier to picture than "Service A" and "Service B."

## Sources

Primary sources cited throughout the principle pages:

- Anthropic — [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- Anthropic — [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Anthropic — [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- Anthropic — [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- Anthropic — [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)
- OpenAI — [Function calling guide](https://developers.openai.com/api/docs/guides/function-calling)

## Related reading

- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents) — the
  sibling document, one layer up (the agent, not the skill).
