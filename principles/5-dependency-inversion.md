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

**Architecture-level — the one that isn't a metaphor at all:** the part of
the system that decides *which* skills apply (discovery, selection, conflict
resolution) should be a pure function of data — it receives an in-memory
representation of the catalog as a parameter and touches no filesystem, no
network, no clock. All the actual I/O happens at the edges of the system and
gets *injected into* that pure core. The high-level policy ("how do we pick
skills") doesn't depend on the low-level detail ("how do we read files");
both depend on a shared, typed abstraction of what a skill and a catalog
look like.

## What frontier labs say

Anthropic's tool-use guidance has a direct, content-level example of this:
when a skill uses an MCP tool, always reference it by a fully-qualified name
— `ServerName:tool_name` — rather than a bare tool name, "to avoid 'tool not
found' errors," because "without the server prefix, Claude may fail to
locate the tool, especially when multiple MCP servers are available." That's
a dependency declared against a stable, namespaced abstraction instead of an
assumption about which concrete server happens to be loaded. The same
guidance separately flags assuming a package or tool is already installed:
`"Use the pdf library to process the file"` is called out as a bad example
precisely because it depends on an implicit, unstated piece of environment
instead of declaring the dependency explicitly (state the install step,
`pip install pypdf`, then use it).
[[source]](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)

## Do / Don't

| Do | Don't |
|---|---|
| Declare skill-to-tool or skill-to-skill dependencies by fully-qualified, stable name, resolved by the composition system. | Reference a bare tool/skill name and hope only one thing in the whole catalog happens to match it. |
| State required packages/tools explicitly, with the install step, inside the skill itself. | Assume a library, binary, or sibling skill is present because it happened to be during authoring. |
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

**Good — dependencies named and declared:**

```markdown
Use the Jira:search_issues tool to look up the ticket.

Install the required package: `pip install pypdf`, then extract the
attachment text:

    from pypdf import PdfReader
    reader = PdfReader("attachment.pdf")
```

Every dependency is a stable, explicit name — a namespaced tool reference or
a declared package — not an assumption about the environment it happened to
be authored on.

## Smell to watch for

Any test for "did we select the right skills" that requires setting up
files on disk, or any instruction that only works because of something true
about one particular machine at one particular moment.
