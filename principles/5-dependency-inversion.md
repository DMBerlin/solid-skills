# D — Dependency Inversion

> High-level modules should not depend on low-level modules; both should
> depend on abstractions. — Robert C. Martin

## Translated for skills — at two different layers

This is the principle most posts flatten into one claim when it's really
two, at two different layers of the system.

**Content-level:** a skill that needs another capability should declare a
dependency on an abstract, stable **name** — "I require the capability that
does X" — resolved by whatever composes skills together, not a hardcoded
path to a specific sibling skill's file or an assumption about which
concrete tool happens to be present.

**Architecture-level — a more literal application, not just a metaphor:**
strictly, dependency inversion says the high-level policy ("how do we pick
skills") should depend on an abstraction, and the low-level detail ("how do
we read files") should depend on that same abstraction — not the other way
around. It does not, by itself, require that policy to be a pure function.
Purity is a separate, complementary choice: if skill selection is written
as a pure function of data — an in-memory catalog goes in, a resolved
selection comes out, nothing inside it touches a filesystem, a network, or
a clock — that's a particularly clean way to *guarantee* the dependency
direction stays inverted, because a pure function has no way to quietly
reach out and depend on a concrete low-level detail even by accident. All
the actual I/O then happens at the edges of the system and gets *injected
into* that core as data. Treat this as a recommended implementation that
makes the inversion easy to verify and easy to test — not as the
definition of the principle itself.

## What frontier labs say

The clearest published guidance here actually documents a different,
related virtue — unambiguous, explicit dependency declaration — which is
*necessary* for inversion but not *sufficient* for it on its own, and it's
worth being precise about the difference. Anthropic's tool-use guidance
recommends that when a skill uses an MCP tool, it reference it by a
fully-qualified name — `ServerName:tool_name` — rather than a bare tool
name, "to avoid 'tool not found' errors," because "without the server
prefix, Claude may fail to locate the tool, especially when multiple MCP
servers are available." Read carefully, that's a concrete binding made
unambiguous — it names one exact server, not an abstraction a composition
layer could swap out later. It solves a real problem (ambiguity,
collisions between same-named tools) but it is not, by itself, dependency
inversion. Genuine inversion would have the skill depend on a *capability*
— "pdf-text-extraction," not "pypdf-mcp-server:extract" — and let whatever
assembles the running system decide which concrete server or library
satisfies it, possibly differently across deployments. The same guidance
separately warns against assuming a package or tool is already installed:
`"Use the pdf library to process the file"` is flagged as a bad example
because it depends on an implicit, unstated piece of environment instead of
declaring the dependency explicitly. That part supports the same design
pressure as inversion — state what you depend on, don't assume it — but it
becomes actual dependency inversion only once the dependency is expressed
as an abstract capability resolved elsewhere, not a named concrete package.
Declaring `pip install pypdf` is real progress (explicit instead of
assumed) but it still names one specific library, not a capability a
composition layer could swap out. Whether runtime installation is even
permitted is a separate, environment-specific policy question: some
runtimes allow it freely; others forbid installing or fetching anything at
runtime without a prior approval step. A skill that just declares "run `pip
install x`" is assuming the former.
[[source]](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

## Do / Don't

| Do | Don't |
|---|---|
| Declare skill-to-skill dependencies by an abstract capability name, resolved by the composition system — not by a specific concrete tool. | Treat a fully-qualified concrete reference (`ServerName:tool_name`) as if it were an abstraction. It's unambiguous, which is good, but it still names one specific implementation. |
| State required packages/tools explicitly, and treat *whether* runtime installation is allowed as a separate, environment-specific policy question. | Assume a library, binary, or sibling skill is present because it happened to be during authoring — or assume install-on-demand is always allowed; some runtimes forbid it or require approval. |
| Write skill-selection logic as a pure function: catalog in, resolved selection out, no I/O inside it. | Let selection logic reach into the filesystem itself — now "did we pick the right skills" can't be unit-tested without mocking disk access. |

## Anti-pattern in practice

**Bad — depends on an unstated, ambient assumption:**

```markdown
Use the search tool to look up the ticket, then use the pdf library to
extract the attachment text.
```

Which `search` tool, of possibly several loaded? Is `pdf` installed, and by
whom? The dependency is real but invisible — it works only by accident, on
whichever machine happens to already have it set up.

**Better, but still concrete — unambiguous, not inverted:**

```markdown
Use the Jira:search_issues tool to look up the ticket, then extract the
attachment text with pypdf (`pip install pypdf` if not already available).
```

This fixes the ambiguity — no more guessing which `search` — and states the
dependency instead of assuming it. That's real progress. But the skill is
still hardcoded to Jira and to pypdf specifically: swap the ticketing system
or the PDF library in a different deployment, and the skill's instructions
have to be rewritten.

**Good — the skill depends on a capability, not a concrete tool:**

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

The skill's instructions never change when the concrete binding does — only
the composition layer's mapping does. That's the actual inversion: the
high-level policy ("what capability is needed") doesn't depend on the
low-level detail ("which vendor or library provides it").

## Smell to watch for

Any test for "did we select the right skills" that requires setting up
files on disk. Also: a skill whose instructions have to be rewritten every
time you swap a vendor or a library — that's a sign the dependency was
concrete and explicit, which is good, but never actually inverted.
