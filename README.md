# SOLID Skills

Design principles for the unit of composition inside an agentic system: the
**skill**, not the whole agent, and not any one vendor's implementation of
it.

[12-Factor Agents](https://github.com/humanlayer/12-factor-agents) covers
how an *agent* should be built: own your prompts, own your context window,
own your control flow. Its factor 10, "Small, Focused Agents," is
essentially Single Responsibility applied to the whole agent.

This repo covers the layer most agent frameworks put underneath that. The
individually invokable **skill** is a discrete capability that gets
discovered, loaded into context, and composed with other skills to build up
a task. That composition layer is exactly the kind of problem
object-oriented design spent decades on. Take the five SOLID principles and
ask what they mean for a skill, and some of it turns out to fit closely,
closer to a literal description of good architecture than a metaphor. Some
of it stretches.

Each principle page says which, backs it with do and don't examples, and
grounds it in what frontier labs have actually published about writing good
skills and tools. Not just frontmatter shape: description quality, body
content, and file structure too.

## Not tied to one framework

Different skill systems express the same underlying ideas with different
mechanics: a closed category taxonomy in one (an adoptable design pattern
illustrated generically here, not a named external spec), a free-text
`description` field in another (Anthropic's actual, published convention),
a folder-per-domain convention in a third. Across the principle pages, more
than one real convention shows up, including Anthropic's own published
Agent Skills spec (`name` and `description` frontmatter, gerund naming,
progressive disclosure) alongside a generic metadata-taxonomy style. The
takeaway is the underlying property, not the syntax that happens to express
it in any one tool.

Not every page carries two sourced conventions to make its point. Liskov
Substitution and Dependency Inversion lean more on illustrative generic
mechanisms, because the published industry conventions for those two are
thinner to begin with. The pages say this plainly instead of papering over
it.

## The five

| # | Principle | One-liner for skills |
|---|---|---|
| S | [Single Responsibility](principles/1-single-responsibility.md) | One skill, one reason to change, one legible trigger. Not three jobs wearing one description. |
| O | [Open/Closed](principles/2-open-closed.md) | New capabilities extend the catalog by being added, tagged, or dropped into a folder. They never require editing what already selects or lists them. |
| L | [Liskov Substitution](principles/3-liskov-substitution.md) | A specialized skill should honor its base's contract. That's provable only where you actually test for structural equivalence; aspirational everywhere else. |
| I | [Interface Segregation](principles/4-interface-segregation.md) | A skill's always-loaded surface should be thin; deeper capability loads on demand, not by default. |
| D | [Dependency Inversion](principles/5-dependency-inversion.md) | Skills depend on an abstract capability resolved elsewhere: not a hardcoded path, not an assumed environment, and not just a concrete reference dressed up as one. |

## Where this bends

SOLID was built for statically typed, compiled, polymorphic code. A skill is
a natural-language artifact, interpreted probabilistically by a model, not
compiled and type-checked.

Several of these five principles have real, buildable, often
test-or-eval-enforceable counterparts once you're managing a catalog of
skills rather than one monolithic prompt. The strength of that evidence
varies, though. Interface Segregation and Single Responsibility are backed
by direct, quoted frontier-lab guidance (progressive disclosure,
unambiguous descriptions) that converges on the same shape independently,
without ever mentioning SOLID by name. Open/Closed and Dependency Inversion
are supported more unevenly and lean on generic illustration where the
published guidance doesn't reach far enough.

Liskov Substitution is the honest exception. None of the sources below
describe anything resembling a general substitutability guarantee across
skill specializations, and that absence is itself evidence for treating it
as the principle that stretches furthest.

## Using this repo

Each principle page has the same shape:

- the classic definition
- what it means once "class" becomes "skill"
- what frontier labs have independently published about it
- a table of dos and don'ts
- worked examples, including an explicit anti-pattern
- a smell to watch for

Examples name real, widely known products (Jira, Asana, BigQuery, JUnit,
GitHub) only as familiar reference points for illustrating a pattern. None
of them tie a principle to one proprietary framework, internal system, or
company's specific architecture. The principle is the point. The product
names are just easier to picture than "Service A" and "Service B."

## Sources

Cited directly, inline, in one or more principle pages:

- Anthropic: [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- Anthropic: [Agent Skills overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
- Anthropic: [Writing effective tools for AI agents](https://www.anthropic.com/engineering/writing-tools-for-agents)
- OpenAI: [Function calling guide](https://developers.openai.com/api/docs/guides/function-calling)

Background reading that shaped this repo's framing but isn't quoted in a
specific page:

- Anthropic: [Equipping agents for the real world with Agent Skills](https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills)
- Anthropic: [Building effective agents](https://www.anthropic.com/engineering/building-effective-agents)

## Related reading

- [12-Factor Agents](https://github.com/humanlayer/12-factor-agents): the
  sibling document, one layer up (the agent, not the skill).
