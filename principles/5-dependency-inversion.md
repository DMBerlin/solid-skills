# D — Dependency Inversion

> High-level modules should not depend on low-level modules; both should
> depend on abstractions. — Robert C. Martin

## Translated for skills — at two different layers

This is the principle most posts flatten into one claim when it's really
two, at two different layers of the system.

**Content-level:** a skill that needs another capability should declare a
dependency on an **abstract capability name** — "I require the capability
that does X" — resolved by whatever composes skills together, not a
hardcoded path to a specific sibling skill's file. The skill author doesn't
need to know, or care, exactly which concrete skill satisfies that name; the
resolution mechanism decides.

**Architecture-level — the one that isn't a metaphor at all:** the part of
the system that decides *which* skills apply (selection, conflict
resolution, precedence) should be a pure function of data — it receives an
in-memory representation of the catalog as a parameter and touches no
filesystem, no network, no clock. All the actual I/O — reading skill files
off disk, writing a resolved result somewhere — happens at the edges of the
system and gets *injected into* that pure core. The high-level policy ("how
do we pick skills") doesn't depend on the low-level detail ("how do we read
files"); both depend on a shared, typed abstraction of what a skill and a
catalog look like.

## Do / Don't

| Do | Don't |
|---|---|
| Declare skill-to-skill dependencies by abstract name (`requires: capability-x`), resolved by the composition system. | Hardcode a path or assume a sibling skill exists at a specific location — now every skill author has to know the whole catalog's layout. |
| Write the skill-selection logic as a pure function: catalog in, resolved selection out, no I/O inside it. | Let the selection logic reach into the filesystem itself — now you can't unit-test "did we pick the right skills" without mocking disk access. |
| Push every read/write to disk to one narrow, designated edge of the system, injected into the pure core as data. | Scatter file-system calls throughout the selection/resolution logic "because it's convenient right here." |

## Worked example

```
loadCatalog(rootDir)      # the only place that touches disk
    -> Catalog             # plain in-memory data

resolve(presetQuery, catalog)   # pure: no fs, no network, no clock
    -> ResolvedSkillSet
```

`resolve` can be unit-tested with a hand-built in-memory catalog and zero
temp directories, mocks, or fixtures on disk. That's not a nice-to-have side
effect of good taste — it's the direct, provable payoff of pushing the
dependency on "how do we get the data" out to the edge and inverting it, so
the high-level policy depends on an abstraction instead of a filesystem.

## Smell to watch for

Any test for "did we select the right skills" that requires setting up files
on disk. That's a sign the selection logic and the I/O layer haven't
actually been inverted — they're still fused.
