# L — Liskov Substitution

> Subtypes must be substitutable for their base types without breaking the
> caller's expectations. — Barbara Liskov

## Translated for skills — and why this one is different from the other four

For compiled code, a type checker proves substitutability before the program
ships. For a skill, there's no equivalent proof: if a specialized skill
*extends* a base discipline, the intent is that anything relying on "the
discipline for this task" gets the same guarantees regardless of which
specialization loaded — the same closing checklist, the same failure
behavior, the same non-negotiable rules — with only the mechanics swapped
out. But a model interpreting that specialized skill can drift from the
base's intent in ways nothing catches at authoring time.

Calling that "solved" would be dishonest. The honest version:
**substitutability is aspirational by default, and becomes real only where
you deliberately test for it.**

## Do / Don't

| Do | Don't |
|---|---|
| If a set of specializations must share a non-negotiable contract (a closing ritual, a reporting format, a hard rule), extract it into one literal, shared block and test that every specialization carries it **byte-for-byte identical**. | Assume that because skill B declares `extends: A`, B behaves consistently with A. Nothing enforces that by construction. |
| Scope your substitutability claims narrowly — "these five skills share this one enforced block" — rather than broadly — "everything that extends anything is substitutable." | Advertise general Liskov compliance across every extension relationship in the catalog. You can't back that claim with a test, so don't make it. |
| Treat a failed byte-identity check as a hard build failure, the same way a broken interface would fail a type check. | Let the shared contract drift silently across specializations because "it's just a prompt, it'll be fine." |

## Worked example

A minimal enforced-equivalence check (conceptual, not framework-specific):

```js
// every "*-tdd" skill must carry the same closing checklist, verbatim
const base = extractBlock('skills/test-driven-development/SKILL.md', 'CLOSING_CHECKLIST');
for (const variant of ['test-driven-development-jvm', 'test-driven-development-kafka']) {
  const block = extractBlock(`skills/${variant}/SKILL.md`, 'CLOSING_CHECKLIST');
  assert.equal(block, base, `${variant} drifted from the base closing checklist`);
}
```

That's a real, mechanically enforced substitutability guarantee — for one
specific block, across one specific set of skills. It is not, and doesn't
claim to be, a guarantee that every `extends` relationship in the catalog is
behaviorally safe to substitute.

## Where this bends

This is the principle where the OOP metaphor is weakest. Use it as a design
goal ("a specialization shouldn't surprise a caller relying on the base's
contract"), and back it with tests only where you can define the contract
precisely enough to check it byte-for-byte or field-for-field. Anywhere you
can't define it that precisely, say so — an untested "should behave like" is
a hope, not a guarantee.
