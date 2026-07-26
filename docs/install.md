# Install

## As a tap

Adding the repository as a tap lets you search and install from it, and pick up updates later:

```bash
hermes skills tap add DKistenev/hermes-inner-life
hermes skills install DKistenev/hermes-inner-life/inner-life
```

## Directly

If you'd rather not add a tap, install the skill from its raw URL:

```bash
hermes skills install https://raw.githubusercontent.com/DKistenev/hermes-inner-life/main/skills/inner-life/SKILL.md
```

Hermes copies `SKILL.md` along with the files it references, so `references/` and `templates/` come with it.

## What lands on disk

```
~/.hermes/skills/inner-life/
├── SKILL.md
├── references/
│   ├── state.md
│   ├── journal.md
│   └── dreams.md
└── templates/state.md
```

## First run

Ask the agent to set up its inner life. It will:

1. Copy `templates/state.md` to `inner-life/state.md` in its working directory
2. Create `inner-life/journal/` and `inner-life/dreams/`

That's the whole setup. There is no config file, no initialization script, and nothing to install alongside it.

## Check that it's there

```bash
hermes skills list
```

`inner-life` should appear in the output. If it doesn't, the tap wasn't added or the install didn't complete — rerun the install command and read its output rather than retrying blind.

## Schedule it

The skill does nothing on its own; the evening and night runs are cron jobs, attached with `--skill inner-life`. See [cron-recipes.md](cron-recipes.md).

## Remove it

```bash
hermes skills uninstall inner-life
```

This removes the skill, not the agent's writing. `inner-life/state.md`, the journal, and the dreams stay in the working directory — delete them separately if you want them gone.
