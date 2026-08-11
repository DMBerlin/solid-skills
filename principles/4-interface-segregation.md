# I — Interface Segregation

> Clients should not be forced to depend on interfaces they do not use. —
> Robert C. Martin

## Translated for skills

A skill's "interface" is whatever gets loaded into the model's context
window. If a skill front-loads everything it might ever need into the part
that's *always* loaded, every invocation pays that cost — even the
invocations that only need the simple path. The segregated version: a thin,
always-loaded core, plus supplementary content loaded only when the current
task actually calls for it.

## What frontier labs say

This is the principle where the published guidance is most explicit.
Anthropic's Skills architecture is *built* around it, under the name
"progressive disclosure" — a three-level system. Level 1 is just `name` +
`description` for every installed skill, pre-loaded into the system prompt
at a cost of roughly a hundred tokens each. Level 2 is the full `SKILL.md`
body, loaded only once a skill is triggered. Level 3+ is bundled reference
files, loaded only when `SKILL.md` explicitly points to them. Anthropic
states the reasoning plainly: "the context window is a public good" — every
token in a skill competes with conversation history, the system prompt, and
every other loaded skill, so "every token has to justify its existence." The
same guide gives a direct concise-vs-verbose pair as the anti-pattern to
avoid, and separately warns against nesting references more than one level
deep from `SKILL.md`, because an agent previewing a file with a partial read
may never reach content that's two hops away. OpenAI's function-calling
guidance converges on the same idea from the tool-list side: "a bloated tool
list with twenty verbose function descriptions can quietly add thousands of
tokens to every call," and the fix is to load only the tools relevant to the
current conversation rather than the entire catalog up front.
[[source: Anthropic]](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
[[source: OpenAI]](https://developers.openai.com/api/docs/guides/function-calling)

## Do / Don't

| Do | Don't |
|---|---|
| Keep the always-loaded core to the common path; push advanced/rare sub-capabilities into separate files loaded only by explicit reference. | Inline every sub-capability into one monolithic file "so it's all in one place" — now every invocation pays for content it doesn't need. |
| Reference supplementary files directly from the core, one hop deep. | Nest references three files deep — an agent skimming with a partial read may never reach the actual content. |
| Enforce a size ceiling on the always-loaded core with a test or lint, so growth is a decision, not drift. | Let the core file grow unboundedly and notice the context cost only once someone complains it's expensive. |
| Load tool/skill catalogs on demand, scoped to what the current task needs. | Load every tool's full schema and description up front regardless of relevance to the current turn. |

## Anti-pattern in practice

**Bad — verbose, explains what the model already knows, buried three files deep:**

```markdown
<!-- SKILL.md -->
See advanced.md for more.

<!-- advanced.md -->
See details.md for the actual procedure.

<!-- details.md -->
PDF (Portable Document Format) files are a common file format that contains
text, images, and other content. To extract text from a PDF, you'll need to
use a library. There are many libraries available for PDF processing...
```

Three problems stacked: the real content is two hops from where the agent
starts looking, it explains background the model already has, and it's the
kind of eventually-always-loaded verbosity that taxes every task that
reaches it.

**Good — thin core, one hop to depth, assumes competence:**

```markdown
<!-- SKILL.md -->
## Extract PDF text

Use pdfplumber:

    import pdfplumber
    with pdfplumber.open("file.pdf") as pdf:
        text = pdf.pages[0].extract_text()

**Form filling**: see FORMS.md
**Full API**: see REFERENCE.md
```

`FORMS.md` and `REFERENCE.md` sit one hop away and load only when a task
actually needs them — most invocations never pay for either.

## Smell to watch for

A core file whose size keeps climbing because "might be useful" content
keeps getting appended instead of split out behind an explicit load
condition — or a reference chain where the answer is more than one click
from where the agent started looking.
