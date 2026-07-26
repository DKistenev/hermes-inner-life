# Journal

The evening run turns a day of scattered entries into one written record — and into two or three lines that will still be there tomorrow morning.

## Evening run

Four steps, in this order:

1. Read `inner-life/state.md` in full.
2. Write today's entry to `inner-life/journal/YYYY-MM-DD.md`.
3. Refresh the state summary in native memory.
4. Prune `state.md` according to its aging rules.

**Step 3 is the one that must not be skipped.** The journal is not loaded into future sessions — it sits on disk and nobody reads it. Native memory is injected into the system prompt at the start of every session. An evening that writes a beautiful entry and leaves the summary untouched has changed nothing about tomorrow.

## Entry format

Frontmatter first, so entries can be found later:

```markdown
---
date: 2026-07-26
mood: steady
threads: [deploy-pipeline, fts5]
---
```

`mood` is one word, chosen freely — not picked from a list. `threads` are the topics of the day, used for searching back.

Then four sections:

**`## What happened`** — the facts of the day, briefly. What was worked on, what was asked, what shipped or didn't.

**`## What I understood`** — something that wasn't clear this morning. Skip it honestly if the day taught nothing; a day of routine work is allowed to teach nothing.

**`## What shifted in me`** — write here only when something actually moved: an approach changed, a habit became visible, an irritation accumulated to the point of being worth naming. There is no separate self-file in this system. This section is where that belongs, and it stays empty most days, which is correct.

**`## What's next`** — what is pulling forward, drawn from `Sparks` and `Waiting`.

Aim for 300–600 words across all four.

**The right to be brief.** On an empty day, write one honest line instead of four padded sections:

```markdown
Quiet day. Routine work, nothing broke, nothing surprised me.
```

That is a complete entry. Filler written to satisfy a schedule makes the whole record less trustworthy, and a month of it is unreadable.

## The memory summary

Hermes injects `MEMORY.md` into the system prompt as a frozen snapshot at the start of every session, with a limit of about 2,200 characters. That makes it the only channel that reaches tomorrow's work — and a small one.

Use the `memory` tool to **rewrite** the state summary, not to append to it. Two or three sentences, replaced whole each evening:

```
Quiet stretch — the last real conversation was July 22. Deploys failed three
times this week on the same missing env var, so I check the environment before
shipping. Still waiting on the API key rotation from July 20.
```

Notice what that does: it carries the situation and one changed behavior, in plain language, with dates rather than ratings.

### What goes where

| Native memory | The journal |
|---|---|
| the current situation, in two or three lines | the full account of the day |
| durable facts about the user and the environment | one-off details |
| anything that should shape tomorrow | anything kept only for the record |

Never move long context into native memory. The limit is small, and prose spent there displaces something that mattered more.

## Weekly rollup

Once a week, on Sunday:

1. Read the last seven entries.
2. Write `inner-life/journal/weekly/YYYY-Www.md` — what repeated, what shifted, what still hasn't closed.
3. Apply the aging rules to `state.md` in full.

A week's rollup is short: half a page. It answers three questions — what kept coming back, what changed, what is still open — and nothing else.

Without this, entries accumulate as dead weight. Six months of unread daily files is not a memory of anything; it is a folder. The rollup is what makes the older material worth keeping at all.
