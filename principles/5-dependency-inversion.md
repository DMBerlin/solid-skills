# D: Dependency Inversion

> High-level modules should not depend on low-level modules; both should
> depend on abstractions. - Robert C. Martin

## Translated for skills, at two different layers

This is the principle most posts flatten into one claim when it's really
two, at two different layers of the system.

At the content level, a skill that needs another capability should declare
a dependency on an abstract, stable name, something like "I require the
capability that does X," resolved by whatever composes skills together.
Not a hardcoded path to a specific sibling skill's file, and not an
assumption about which concrete tool happens to be present.

At the architecture level, this is a more literal application, not a
metaphor. Strictly, dependency inversion says the high-level policy ("how
do we pick skills") should depend on an abstraction, and the low-level
detail ("how do we read files") should depend on that same abstraction, not
the other way around. It does not, by itself, require that policy to be a
pure function.

Purity is a separate, complementary choice. If skill selection is written
as a pure function of data (an in-memory catalog goes in, a resolved
selection comes out, nothing inside it touches a filesystem, a network, or
a clock), that's a particularly clean way to guarantee the dependency
direction stays inverted: a pure function has no way to quietly reach out
and depend on a concrete low-level detail, even by accident. All the
actual I/O then happens at the edges of the system and gets injected into
that core as data. Treat this as a recommended implementation that makes
the inversion easy to verify and easy to test, not as the definition of
the principle itself.

## What frontier labs say

The clearest published guidance here actually documents a different,
related virtue: unambiguous, explicit dependency declaration. That's
necessary for inversion but not sufficient for it on its own, and it's
worth being precise about the difference.

Anthropic's tool-use guidance recommends that when a skill uses an MCP
tool, it reference it by a fully-qualified name, `ServerName:tool_name`,
rather than a bare tool name, "to avoid 'tool not found' errors," because
"without the server prefix, Claude may fail to locate the tool, especially
when multiple MCP servers are available." Read carefully, that's a
concrete binding made unambiguous. It names one exact server, not an
abstraction a composition layer could swap out later. It solves a real
problem, ambiguity and collisions between same-named tools, but it is not,
by itself, dependency inversion. Genuine inversion would have the skill
depend on a capability, "pdf-text-extraction," not
"pypdf-mcp-server:extract," and let whatever assembles the running system
decide which concrete server or library satisfies it, possibly differently
across deployments.

The same guidance separately warns against assuming a package or tool is
already installed. `"Use the pdf library to process the file"` is flagged
as a bad example because it depends on an implicit, unstated piece of
environment instead of declaring the dependency explicitly. That part
supports the same design pressure as inversion: state what you depend on,
don't assume it. But it becomes actual dependency inversion only once the
dependency is expressed as an abstract capability resolved elsewhere, not a
named concrete package. Declaring `pip install pypdf` is real progress
(explicit instead of assumed), but it still names one specific library,
not a capability a composition layer could swap out. Whether runtime
installation is even permitted is a separate, environment-specific policy
question. Some runtimes allow it freely. Others forbid installing or
fetching anything at runtime without a prior approval step. A skill that
just declares "run `pip install x`" is assuming the former.
[[source]](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

## Do and don't

| Do                                                                                                                                                 | Don't                                                                                                                                                                                                  |
|----------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Declare skill-to-skill dependencies by an abstract capability name, resolved by the composition system, not by a specific concrete tool.           | Treat a fully-qualified concrete reference (`ServerName:tool_name`) as if it were an abstraction. It's unambiguous, which is good, but it still names one specific implementation.                     |
| State required packages/tools explicitly, and treat *whether* runtime installation is allowed as a separate, environment-specific policy question. | Assume a library, binary, or sibling skill is present because it happened to be during authoring. Or assume install-on-demand is always allowed: some runtimes forbid it or require approval.          |
| Write skill-selection logic as a pure function: catalog in, resolved selection out, no I/O inside it.                                              | Let selection logic reach into the filesystem itself. Now "did we pick the right skills" gets harder to test fast and in isolation: you either mock disk access or pay for real file I/O on every run. |

## Anti-pattern in practice

Here's a dependency on an unstated, ambient assumption:

```markdown
Use the search tool to look up the ticket, then use the pdf library to
extract the attachment text.
```

The dependency is real but invisible: it's not clear which `search` tool
this refers to, of possibly several loaded, or whether `pdf` is installed
and by whom. It works only by accident, on whichever machine happens to
already have it set up.

Here's an improvement that's still concrete: unambiguous, but not
inverted:

```markdown
Use the Jira:search_issues tool to look up the ticket, then extract the
attachment text with pypdf (`pip install pypdf` if not already available).
```

This fixes the ambiguity (no more guessing which `search`) and states the
dependency instead of assuming it. That's real progress. But the skill is
still hardcoded to Jira and to pypdf specifically. Swap the ticketing
system or the PDF library in a different deployment, and the skill's
instructions have to be rewritten.

Here's the actual fix: the skill depends on a capability, not a concrete
tool:

```markdown
Look up the ticket using this environment's configured ticketing
capability, then extract the attachment text using this environment's
configured PDF-extraction capability.
```

```yaml
# resolved by the composition layer, not authored into the skill:
capabilities:
  ticketing: Jira:search_issues
  pdf-extraction: pypdf-mcp:extract_text
```

The skill's instructions never change when the concrete binding does. Only
the composition layer's mapping does. That's the actual inversion: the
high-level policy ("what capability is needed") doesn't depend on the
low-level detail ("which vendor or library provides it").

If a required capability isn't configured for this environment, the skill
should stop and report exactly which binding is missing, instead of
guessing at a tool, silently skipping the step, or reaching for a package
installer on its own. A missing binding is a configuration gap for
whoever owns the composition layer to fix, not something the skill should
improvise around.

## Smell to watch for

A test for "did we select the right skills" that has to set up real files
on disk first is a smell, not proof the design is broken. It's evidence
the policy is coupled to I/O, which makes tests slower and more brittle
than a pure version would be. Also, a skill whose instructions have to be
rewritten every time you swap a vendor or a library is a sign the
dependency was concrete and explicit, which is good, but never actually
inverted.
