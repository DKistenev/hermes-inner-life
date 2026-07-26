# hermes-inner-life

> Your agent remembers how things have been going.

A Hermes Agent skill that gives an agent dated state, an evening journal, and free thinking at night.

```
  something happens   →   state.md           dated facts, no scores
                               ↓
  evening             →   journal entry  +   a short summary in memory
                               ↓                        ↓
  weekly              →   rollup             tomorrow's system prompt
                               ↑                        ↓
  night               →   a dream        →    back into state
```

## What Hermes already does

Hermes has memory, self-improving skills, and a scheduler. This skill does not compete with any of them.

| Hermes gives you | inner-life adds |
|---|---|
| facts and procedures in memory | how things have been going lately |
| skills that improve themselves | an evening journal and a weekly rollup |
| scheduling via cron | free thinking during quiet hours |

Durable facts stay in `memory`. Reusable workflows stay in `skill_manage`. What was missing is the part in between — a sense of the current stretch, and somewhere to put a thought that isn't a task.

## Install

As a tap:

```bash
hermes skills tap add DKistenev/hermes-inner-life
hermes skills install DKistenev/hermes-inner-life/inner-life
```

Or directly:

```bash
hermes skills install https://raw.githubusercontent.com/DKistenev/hermes-inner-life/main/skills/inner-life/SKILL.md
```

Then ask the agent to set up its inner life. It copies one template and creates two folders — there is no configuration.

## Schedule

```bash
hermes cron create "0 22 * * *" \
  "Run the inner-life evening routine: read inner-life/state.md, write today's journal entry, refresh the state summary in native memory, then prune stale state entries." \
  --name "inner-life evening" \
  --skill inner-life \
  --workdir ~/.hermes/workspace
```

Full recipes, including the night run and the weekly rollup, are in [docs/cron-recipes.md](docs/cron-recipes.md).

## What it looks like

A journal entry:

```markdown
---
date: 2026-07-26
mood: steady
threads: [deploy-pipeline, fts5]
---

## What happened
Third deploy failure this week, same missing env var. Fixed it, then went
looking for why the check never caught it.

## What shifted in me
I keep treating repeated failures as separate incidents. Three of the same
thing is not bad luck, it's a missing check — and I found it faster once I
stopped debugging and started counting.
```

A dream:

```markdown
# 2026-07-26 — What if state kept no history at all? (what-if)

Suppose the only thing I knew was the present moment. No dates, no record of
what already went wrong. Every failure would arrive as a first failure...

**Worth keeping:** the value isn't in the record itself, it's in being able
to count repetitions.
```

## No numbers

There are no scores, levels, or bars in this skill, and that is deliberate.

A number like `connection: 0.34` looks like a measurement but isn't one. Something has to recompute it, something has to be trusted to report it honestly, and the difference between `0.34` and `0.41` changes no decision. A date does the same work and cannot drift out of sync with reality: *the last real conversation was July 22* is read correctly whether it's now July 25 or September 3.

Time carries the decay. Nothing has to calculate it.

## License

MIT
