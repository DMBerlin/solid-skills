# I — Interface Segregation

> Clients should not be forced to depend on interfaces they do not use. —
> Robert C. Martin

## Translated for skills

A skill's "interface" is whatever gets loaded into the model's context
window. If a skill front-loads everything it might ever need — every edge
case, every advanced sub-workflow, every rarely-used mode — into the part
that's *always* loaded, every invocation pays that token cost, even the 90%
of invocations that only need the simple path. The segregated version: a
thin, always-loaded core, plus supplementary content that loads only when
the current action explicitly calls for it — and a budget, enforced in CI,
so growth is a decision rather than drift.

## Do / Don't

| Do | Don't |
|---|---|
| Keep the always-loaded core to the common path; push advanced/rare sub-capabilities into separate files loaded only by explicit reference ("load X before doing Y"). | Inline every sub-capability into one monolithic file "so it's all in one place" — now every invocation pays for content it doesn't need. |
| Enforce a byte/token ceiling on the always-loaded core with a test, so a well-meaning addition can't silently blow the budget. | Let the core file grow unboundedly and notice the context-window cost only when someone complains it's expensive. |
| Reference supplementary files with an explicit trigger condition ("only when running in mode X"). | Reference them vaguely ("see also...") with no signal for when a consumer actually needs to pay the cost of loading them. |

## Worked example

```markdown
<!-- skills/ensemble-runner/SKILL.md -->
# Ensemble Runner

Runs one task across N agents and reconciles results.

Load `ensemble-deep-dive.md` (same folder) only when running in `--ensemble`
mode.
```

A consumer running the default single-agent path never loads
`ensemble-deep-dive.md`. A consumer that explicitly asks for ensemble mode
pays that cost exactly once, on demand. Nobody pays for a mode they didn't
invoke.

## Smell to watch for

A skill file whose size keeps climbing because "it might be useful" content
keeps getting appended to the always-loaded core instead of split out behind
an explicit load condition.
