# L: Liskov Substitution

> Subtypes must be substitutable for their base types without breaking the
> caller's expectations. - Barbara Liskov

## Translated for skills, and why this one differs from the rest

For compiled code, a type checker can catch structural incompatibility
before the program ships: a subtype method with an incompatible signature,
say. It generally cannot catch behavioral incompatibility. The textbook
case is a `Square` that subtypes `Rectangle` by keeping width and height
locked together. It type-checks perfectly, and still breaks any code that
assumes setting `width` leaves `height` alone. Structural compatibility was
always necessary for substitutability, never sufficient, even in strictly
typed languages.

For a skill, even a system that validates frontmatter or schema shape at
load time is only checking that the skill is well-formed (that a category
tag is one of a closed set, say), not that a specialization honors the
base's interface or behavior. That's a real structural check. It just
doesn't reach behavioral substitutability.

If a specialized skill extends, or is meant to stand in for, a base
capability, the intent is that anything relying on "the discipline for this
task" gets the same guarantees regardless of which specialization loaded:
the same closing checklist, the same failure behavior, the same
non-negotiable rules, with only the mechanics swapped out. But a model
interpreting that specialized skill can drift from the base's intent in
ways nothing catches at authoring time.

Calling that "solved" would be dishonest. The honest version:
**substitutability is aspirational by default, and has enforceable
evidence only where you deliberately test a defined contract.**

## What frontier labs say (or rather, don't)

This is the one principle where a look at the published guidance comes up
mostly empty. Anthropic's [authoring
guide](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
talks at length about descriptions, structure, progressive disclosure, and
evaluation-driven iteration, but nowhere addresses whether a specialized
skill behaves consistently with the general one it specializes. OpenAI's
[function-calling
guidance](https://developers.openai.com/api/docs/guides/function-calling)
is silent on it too.

None of the sources reviewed for this repo describe anything resembling a
general substitutability guarantee for skill specializations. That
absence, across every source checked, says something on its own. The
closest anyone gets in practice is testing one specific, narrow contract
for equality, which is a much smaller, much more honest claim than
substitutability.

## Do and don't

| Do | Don't |
|---|---|
| If a set of specializations must share a non-negotiable contract (a closing ritual, a reporting format, a hard rule), extract it into one literal, shared block and test that every specialization carries it byte-for-byte identical. | Assume that because skill B extends or specializes skill A, B behaves consistently with A. Nothing enforces that by construction. |
| Scope your substitutability claims narrowly (for example, "these five skills share this one enforced block") rather than broadly ("everything that extends anything is substitutable"). | Advertise general Liskov compliance across every extension relationship in the catalog. You can't back that claim with a test, so don't make it. |
| Treat a failed equality check as a hard build failure, the same way a broken interface would fail a type check. | Let the shared contract drift silently across specializations because "it's just a prompt, it'll be fine." |

## Anti-pattern in practice

Here's silent drift between a base and its specialization:

```markdown
<!-- discipline/testing.md -->
## Closing checklist
- [ ] All tests pass
- [ ] No skipped tests without a linked ticket
- [ ] Coverage report attached
```

```markdown
<!-- stack/testing-jvm.md (extends discipline/testing.md) -->
## Closing checklist
- [ ] Tests pass
- [ ] Coverage looks reasonable
```

Nothing enforces that these say the same thing, and they don't. "No
skipped tests without a linked ticket" quietly disappeared. "Coverage
report attached" softened into "looks reasonable." A caller trusting the
base's contract gets a silently weaker one from the specialization.

Here's the fix: the shared contract as one literal block, checked for
equality:

```markdown
<!-- shared/closing-checklist.md: included verbatim by every specialization -->
## Closing checklist
- [ ] All tests pass
- [ ] No skipped tests without a linked ticket
- [ ] Coverage report attached
```

```js
// build check: every specialization must include this block byte-for-byte
assert.equal(extractBlock(variantPath, 'Closing checklist'), sharedBlock);
```

## Where this bends

This is the principle where the OOP metaphor is weakest. Use it as a design
goal ("a specialization shouldn't surprise a caller relying on the base's
contract"), and back it with tests only where you can define the contract
precisely enough to check it byte-for-byte or field-for-field. Anywhere you
can't define it that precisely, say so. An untested "should behave like"
is a hope, not a guarantee.
