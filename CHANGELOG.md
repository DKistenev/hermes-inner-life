# Changelog

## 1.1.0 — 2026-07-27

Answers a registry security review. No change to what the skill does — changes to when it starts and what it is allowed to write down.

- Triggers are explicit: a scheduled run or a direct request, never the skill's own judgment that a conversation was worth recording
- First-time setup asks before creating anything; a missing `state.md` means the skill was never turned on
- New `What this writes down` section — where each kind of writing lands, which one reaches every later session, and what uninstalling leaves behind
- Data minimisation rules across state, journal, and dreams: record that something happened, not what was said in it
- Credentials, personal details, and third parties are out of the memory summary, which is the only thing here that travels between sessions
- The skill body no longer names a platform, so it reads the same on any runtime

## 1.0.0 — 2026-07-26

First release.

- One skill: state, an evening journal, and dreams at night
- State is dated facts, not scores — time carries the decay
- The journal absorbs reflection; there is no separate self file
- Dreams check the last two weeks so they don't repeat themselves
- Behavior changes through the native memory summary, not through rules
- No scripts, no dependencies
