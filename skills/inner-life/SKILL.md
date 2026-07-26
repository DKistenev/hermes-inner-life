---
name: inner-life
description: Use when the agent should keep a sense of itself across sessions — logging what happens, writing an evening journal, or thinking freely at night. Keeps dated state that carries forward and colors how the agent works the next day.
version: 1.0.0
author: DKistenev
license: MIT
metadata:
  hermes:
    tags: [journal, dreams, self-reflection, continuity, memory]
---

# Inner Life

## Overview

An agent accumulates a great deal in a day and keeps almost none of it. Facts survive, procedures survive — but how things have been going does not. Every morning starts the same way regardless of whether yesterday was a good day or a rough one.

This skill adds three things: **state**, a running record of dated facts about how things are going; an **evening journal** that turns the day into a written entry; and **dreams**, free thinking during the quiet hours. Tomorrow is reached through a short summary written into native memory, which Hermes injects into the system prompt of every session. There are no scores anywhere in this — how long ago something happened carries the meaning that a number would only obscure.

## When to Use

- The evening run is due — write the day's entry and refresh the summary
- The night run is due — think about something that isn't a task
- Something happened worth recording: a conversation, a failure, a thing that caught attention
- The user asks for the journal, a weekly rollup, or what's been going on

**Don't use for:** storing facts or procedures. Hermes has `memory` for durable facts and `skill_manage` for reusable workflows, and its background review already maintains both. This skill does not duplicate either.

## Three modes

| Mode | When | Read |
|---|---|---|
| Logging | something happened | `references/state.md` |
| Evening | once a day | `references/journal.md` |
| Night | quiet hours | `references/dreams.md` |

Read only the reference for the mode at hand. All three are never needed at once.

## First run

If `inner-life/state.md` does not exist:

1. Copy `templates/state.md` to `inner-life/state.md` in the working directory.
2. Create `inner-life/journal/` and `inner-life/dreams/`.

Nothing else is needed — there is no configuration and no setup script.

## What this is not

Not a memory system: Hermes already has one, and this skill deliberately stays out of it.

Not mood tracking. There are no levels, percentages, or bars here on purpose. A date already says everything a rating would claim to say, it cannot drift out of sync with reality, and it requires nobody to be trusted with keeping it accurate.

## Common Pitfalls

1. **Writing the journal and skipping the memory summary.** The journal never enters a future session's context. Only the summary does. Skip it and tomorrow notices nothing.
2. **Recording judgments instead of facts.** `getting frustrated with deploys` is useless in a month; `third failed deploy, same missing env var` is not.
3. **Dreaming without checking the last two weeks.** The same preoccupation comes back in new words, and the record loses its value.
4. **Filling native memory with prose.** The limit is about 2,200 characters; long text there displaces something that mattered more.
5. **Writing to satisfy the schedule.** An empty day deserves one honest line, and an empty night deserves no file at all.

## Verification Checklist

- [ ] `inner-life/state.md` exists and its entries are dated
- [ ] Today's journal entry exists, or the day was deliberately left blank
- [ ] The native memory summary reflects recent dates, not last month's situation
- [ ] Stale state entries have been pruned, and anything still open remains in `Waiting`
