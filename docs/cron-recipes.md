# Cron recipes

Three jobs. The evening one is the only one that is genuinely required — without it the state file grows and nothing ever reaches tomorrow.

Every recipe below attaches the skill explicitly with `--skill inner-life`, so the run doesn't depend on the prompt happening to match. `--workdir` sets where `inner-life/` lives; point it at the directory the agent actually works from.

## Evening

```bash
hermes cron create "0 22 * * *" \
  "Run the inner-life evening routine: read inner-life/state.md, write today's journal entry, refresh the state summary in native memory, then prune stale state entries." \
  --name "inner-life evening" \
  --skill inner-life \
  --workdir ~/.hermes/workspace
```

Set the hour to when your day actually ends. An evening run at 22:00 that fires while you're still working produces an entry about half a day.

## Night

```bash
hermes cron create "0 3 * * *" \
  "Run the inner-life night routine: check the last two weeks of dreams, pick a topic that has not come up, think freely about it, and add anything worth keeping to Sparks in inner-life/state.md." \
  --name "inner-life dream" \
  --skill inner-life \
  --workdir ~/.hermes/workspace
```

Pick an hour when nothing else is scheduled. Some nights this job will correctly produce no file at all.

## Weekly

```bash
hermes cron create "0 21 * * 0" \
  "Run the inner-life weekly rollup: fold the last seven journal entries into one weekly note under inner-life/journal/weekly/, then age out stale state entries." \
  --name "inner-life weekly" \
  --skill inner-life \
  --workdir ~/.hermes/workspace
```

Sunday evening, before the evening run of the same day, so the week closes cleanly.

## Delivery

By default the runs are silent. To have the evening summary land in a channel, add a delivery target:

```bash
--deliver telegram
```

Valid targets are `origin`, `local`, `telegram`, `discord`, `signal`, or `platform:chat_id`.

Consider leaving the night run undelivered. A dream is for the record, not for a notification at 3 a.m.

## Checking on the jobs

```bash
hermes cron list
hermes cron run <id>     # trigger on the next tick, to test without waiting
```

Run each job once manually after creating it. It is better to find out that the evening prompt produces a thin entry now than to discover it after a month of thin entries.
