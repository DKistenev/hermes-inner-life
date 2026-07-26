# hermes-inner-life v1.0 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Собрать с нуля Hermes-скилл `inner-life`, дающий агенту внутреннее состояние, вечерний дневник и ночные сны, и опубликовать его вместо содержимого репозитория `DKistenev/openclaw-inner-life`.

**Architecture:** Один markdown-скилл без скриптов и зависимостей. Состояние — журнал фактов с датами (`inner-life/state.md` в рабочей директории агента), затухание берётся из времени. Вечерний прогон пишет дневник и обновляет короткую сводку в нативной памяти Hermes, которая вклеивается в системный промпт каждой сессии — это единственный канал влияния на поведение. Ночной прогон пишет сны и возвращает находки в состояние.

**Tech Stack:** Markdown + YAML frontmatter. Никакого кода. Проверки — команды `grep`/`python3` из стандартной поставки macOS.

## Global Constraints

- **Всё, что попадает в `main`, — на английском.** Без исключений: `README.md`, `CHANGELOG.md`, `skills/**`, `docs/install.md`, `docs/cron-recipes.md`.
- Рабочие документы на русском (`docs/superpowers/specs/`, `docs/superpowers/plans/`) в `main` не публикуются — они уходят на отдельную ветку `planning`, по общей конвенции автора. См. Task 8, шаг 3.
- **Ноль скриптов** внутри `skills/` и ноль внешних зависимостей (`jq`, `bc`, `python` в рантайме скилла запрещены).
- **Никаких числовых метрик состояния.** Ни шкал, ни коэффициентов, ни порогов вида `> 0.7`. Даты — можно.
- **Никаких упоминаний** `OpenClaw`, `clawhub`, `clawdbot`, `SOUL.md`-протоколов OpenClaw внутри `skills/`. В `README.md` допускается ровно одна строка со ссылкой на тег `v1.0-openclaw`.
- Структура тапа: `skills/<имя-скилла>/SKILL.md`, плоско, без категорий в путях. Эталон — `anthropics/skills`.
- Frontmatter скилла: `name` в нижнем регистре через дефис, не длиннее 64 символов; `description` строго в формате `Use when <trigger>. <one-line behavior>.`
- Все пути внутри скилла — относительные (`inner-life/state.md`), никаких `~/.hermes/...` кроме упоминания нативной памяти.
- Рабочая директория: `~/hermes-inner-life`. Пуш в удалённый репозиторий — только в задачах 8 и 9, после явного подтверждения пользователя.

---

### Task 1: Скелет репозитория

**Files:**
- Create: `LICENSE`
- Create: `.gitignore`
- Create: `skills/inner-life/references/.gitkeep`
- Create: `skills/inner-life/templates/.gitkeep`

**Interfaces:**
- Produces: каталог `skills/inner-life/` с подпапками `references/` и `templates/` — в него пишут задачи 2–5.

- [ ] **Step 1: Создать структуру каталогов**

```bash
cd ~/hermes-inner-life
mkdir -p skills/inner-life/references skills/inner-life/templates docs
touch skills/inner-life/references/.gitkeep skills/inner-life/templates/.gitkeep
```

- [ ] **Step 2: Написать LICENSE**

Файл `LICENSE` — стандартный текст MIT, строка копирайта:

```
Copyright (c) 2026 Daniil Kistenev
```

Остальной текст — каноническая лицензия MIT без изменений.

- [ ] **Step 3: Написать .gitignore**

```
.DS_Store
*.swp
```

- [ ] **Step 4: Проверить, что структура совпадает с эталонным тапом**

Run:
```bash
cd ~/hermes-inner-life && find skills -type d | sort
```
Expected: ровно три строки — `skills`, `skills/inner-life`, `skills/inner-life/references`, `skills/inner-life/templates`. Никаких категорий между `skills/` и `inner-life/`.

- [ ] **Step 5: Commit**

```bash
cd ~/hermes-inner-life
git add LICENSE .gitignore skills/
git commit -m "chore: repository skeleton"
```

---

### Task 2: Состояние — шаблон и правила

**Files:**
- Create: `skills/inner-life/templates/state.md`
- Create: `skills/inner-life/references/state.md`

**Interfaces:**
- Produces: четыре имени секций — `Contact`, `Friction`, `Sparks`, `Waiting`. Задачи 3, 4 и 5 ссылаются на них дословно. Формат строки записи: `- YYYY-MM-DD — <текст>`.

- [ ] **Step 1: Написать шаблон состояния**

Файл `skills/inner-life/templates/state.md` — это то, что агент копирует к себе при первом запуске:

```markdown
# Inner state

Facts with dates. No scores, no scales — how long ago it happened is the signal.

## Contact
When we last talked and what it felt like.

- 2026-07-22 — long call about the deploy pipeline. Unhurried, he was thinking out loud.

## Friction
Where things went wrong recently.

- 2026-07-25 — third failed deploy this week, same missing env var.

## Sparks
What caught my attention and is worth digging into.

- 2026-07-24 — how FTS5 ranks results. Want to understand the scoring.

## Waiting
Sent, asked, or promised — still open.

- 2026-07-20 — asked about rotating the API key. No answer yet.
```

- [ ] **Step 2: Написать правила работы с состоянием**

Файл `skills/inner-life/references/state.md`. Обязательные разделы и их содержание:

**`## Reading state`** — центральное правило: значение несёт не количество записей, а их давность. Три дня без Contact и три недели без Contact — разные состояния, и модель обязана читать разницу по датам, а не искать оценку. Прямо запретить выводить любые числовые «уровни».

**`## Writing state`** — одна запись = одна строка формата `- YYYY-MM-DD — <текст>`. Записывать по ходу дела, не откладывая до вечера. Текст — конкретный факт, а не оценка: «третий провал деплоя, тот же env var» вместо «фрустрация растёт».

**`## Which section`** — таблица с колонками Section / What goes here / Example:

| Section | What goes here | Example |
|---|---|---|
| Contact | разговоры с пользователем, их тон | `- 2026-07-22 — long call about deploys, unhurried` |
| Friction | сбои, повторяющиеся ошибки, тупики | `- 2026-07-25 — third failed deploy, same env var` |
| Sparks | что зацепило и хочется копнуть | `- 2026-07-24 — FTS5 ranking, want to understand scoring` |
| Waiting | отправленное и обещанное без ответа | `- 2026-07-20 — asked about key rotation, no answer` |

**`## Aging`** — правила устаревания, применяются при вечерней уборке (Task 3):

| Section | Keep |
|---|---|
| Contact | последние 5 записей |
| Friction | 30 дней |
| Sparks | 14 дней, либо до попадания в дневник |
| Waiting | до закрытия — дата не удаляет запись, её снимает только ответ или отмена |

**`## Don't`** — не заводить новых секций; не писать числовых оценок; не превращать записи в прозу; не дублировать то, что уже ушло в нативную память.

- [ ] **Step 3: Проверить отсутствие числовых метрик**

Run:
```bash
cd ~/hermes-inner-life && grep -nE '[0-9]+\.[0-9]+|\b(0\.[0-9]|score|level|scale|threshold)\b' skills/inner-life/templates/state.md skills/inner-life/references/state.md
```
Expected: пустой вывод. Любое совпадение — нарушение Global Constraints, кроме дат вида `2026-07-22` (они не содержат точки и под шаблон не подпадают).

- [ ] **Step 4: Проверить единообразие формата строк**

Run:
```bash
cd ~/hermes-inner-life && grep -n '^- ' skills/inner-life/templates/state.md
```
Expected: каждая строка вида `- YYYY-MM-DD — текст` с длинным тире. Строк ровно четыре, по одной на секцию.

- [ ] **Step 5: Commit**

```bash
cd ~/hermes-inner-life
git add skills/inner-life/templates/state.md skills/inner-life/references/state.md
git rm --cached skills/inner-life/templates/.gitkeep 2>/dev/null; rm -f skills/inner-life/templates/.gitkeep
git add -A skills/inner-life/templates
git commit -m "feat: state format — dated facts, no scores"
```

---

### Task 3: Дневник, сводка и недельная свёртка

**Files:**
- Create: `skills/inner-life/references/journal.md`

**Interfaces:**
- Consumes: секции состояния из Task 2 (`Contact`, `Friction`, `Sparks`, `Waiting`), правила устаревания из `## Aging`.
- Produces: путь `inner-life/journal/YYYY-MM-DD.md`, фронтматтер с полями `date`, `mood`, `threads`; путь свёртки `inner-life/journal/weekly/YYYY-Www.md`. Task 5 и Task 7 ссылаются на эти пути дословно.

- [ ] **Step 1: Написать раздел вечернего прогона**

Файл `skills/inner-life/references/journal.md`, раздел `## Evening run` — четыре шага строго в этом порядке:

1. Прочитать `inner-life/state.md` целиком.
2. Написать `inner-life/journal/YYYY-MM-DD.md`.
3. Обновить сводку состояния в нативной памяти.
4. Почистить `state.md` по правилам `## Aging`.

Явно указать: шаг 3 нельзя пропускать — без него запись остаётся мёртвой, потому что дневник в контекст следующей сессии не попадает.

- [ ] **Step 2: Написать формат записи**

Раздел `## Entry format`. Фронтматтер дословно:

```markdown
---
date: 2026-07-26
mood: steady
threads: [deploy-pipeline, fts5]
---
```

Поле `mood` — одно слово свободной формы, не из фиксированного списка. Поле `threads` — темы дня для последующего поиска.

Четыре секции записи:

- `## What happened` — факты дня, коротко.
- `## What I understood` — то, чего не знал утром.
- `## What shifted in me` — сюда втекает рефлексия. Пишется только когда действительно есть сдвиг: изменился подход, всплыла собственная привычка, накопилось раздражение. Отдельного `SELF.md` в системе нет.
- `## What's next` — что тянет вперёд, из `Sparks` и `Waiting`.

Объём — 300–600 слов. Право промолчать: если день пустой, пишется одна честная строка вместо четырёх пустых секций, и это нормально.

- [ ] **Step 3: Написать контракт с нативной памятью**

Раздел `## The memory summary`. Это главный раздел файла.

Объяснить механику: Hermes вклеивает `MEMORY.md` в системный промпт снимком в начале каждой сессии, лимит — 2200 символов. Поэтому сводка обновляется инструментом `memory`, занимает две-три строки и переписывается целиком, а не дописывается.

Дать образец сводки:

```
Quiet stretch — last real conversation was July 22. Deploys failed three
times this week on the same env var, so I double-check the environment
before shipping. Waiting on the API key rotation since July 20.
```

Таблица разделения:

| Goes to native memory | Goes to the journal |
|---|---|
| текущее состояние в двух-трёх строках | полный рассказ о дне |
| устойчивые факты о пользователе и среде | разовые детали |
| то, что должно влиять на завтра | то, что нужно только для истории |

Явное правило: длинный контекст в нативную память не писать — лимит съест то, что важнее.

- [ ] **Step 4: Написать недельную свёртку**

Раздел `## Weekly rollup`, воскресный прогон:

1. Прочитать семь последних записей.
2. Написать `inner-life/journal/weekly/YYYY-Www.md` — что за неделю повторялось, что сдвинулось, что так и не закрылось.
3. Применить `## Aging` к `state.md` целиком.

Обоснование включить в текст: без свёртки записи копятся мёртвым грузом, и через полгода их никто, включая самого агента, не читает.

- [ ] **Step 5: Проверить связность ссылок на секции состояния**

Run:
```bash
cd ~/hermes-inner-life && grep -oE '\b(Contact|Friction|Sparks|Waiting)\b' skills/inner-life/references/journal.md | sort -u
```
Expected: имена секций совпадают с Task 2 посимвольно. Ни `Frictions`, ни `Spark`.

- [ ] **Step 6: Проверить отсутствие числовых метрик и запрещённых слов**

Run:
```bash
cd ~/hermes-inner-life && grep -niE 'clawhub|openclaw|clawdbot|SELF\.md|[0-9]+\.[0-9]+' skills/inner-life/references/journal.md
```
Expected: пустой вывод. Упоминание `SELF.md` означает, что рефлексию не влили в дневник, как решено в спеке.

- [ ] **Step 7: Commit**

```bash
cd ~/hermes-inner-life
git add skills/inner-life/references/journal.md
git commit -m "feat: evening journal, memory summary, weekly rollup"
```

---

### Task 4: Сны

**Files:**
- Create: `skills/inner-life/references/dreams.md`

**Interfaces:**
- Consumes: секцию `Sparks` из Task 2.
- Produces: путь `inner-life/dreams/YYYY-MM-DD.md`. Task 5 и Task 7 ссылаются на него дословно.

- [ ] **Step 1: Написать раздел ночного прогона**

Файл `skills/inner-life/references/dreams.md`, раздел `## Night run`:

1. Прочитать `inner-life/state.md` — что сейчас занимает.
2. Прочитать заголовки снов за последние 14 дней.
3. Выбрать тему, которой в этих заголовках нет.
4. Думать свободно и записать в `inner-life/dreams/YYYY-MM-DD.md`.
5. Если родилось что-то стоящее — добавить строку в `Sparks`.

Шаг 2 обязателен: без сверки агенту снится одно и то же, и записи обесцениваются за неделю.

- [ ] **Step 2: Написать жанры**

Раздел `## Kinds of dreams` — таблица из пяти жанров:

| Kind | The question it asks |
|---|---|
| future | What could this become in a year? |
| tangent | Something adjacent I keep not looking at |
| connection | Two unrelated things that might rhyme |
| what-if | A constraint removed, or reversed |
| retrospective | An old decision, seen from now |

Не расширять список без нужды: чем шире меню, тем более случайным выходит выбор.

- [ ] **Step 3: Написать формат записи**

Раздел `## Entry format`:

```markdown
# 2026-07-26 — What if state had no history at all? (what-if)

[Free thinking. Not a report, not a plan — thinking out loud, written down.
Follow the thought where it goes.]

**Worth keeping:** one sentence, or nothing.
```

Объём — 300–500 слов, одна мысль до конца, а не пять по касательной.

- [ ] **Step 4: Написать правило молчания**

Раздел `## The right to skip`. Если сказать нечего — файл не создаётся вовсе. Пустой сон хуже отсутствующего: он засоряет сверку тем и приучает писать ради строчки. Явно указать: пропуск не считается сбоем прогона.

- [ ] **Step 5: Проверить связность и отсутствие скриптов**

Run:
```bash
cd ~/hermes-inner-life && grep -niE 'should-dream|\.sh\b|random|dice|probability|jq |clawhub|openclaw' skills/inner-life/references/dreams.md
```
Expected: пустой вывод. Рандомный гейт прототипа не должен вернуться — расписание держит нативный cron.

- [ ] **Step 6: Проверить ссылку на Sparks**

Run:
```bash
cd ~/hermes-inner-life && grep -c 'Sparks' skills/inner-life/references/dreams.md
```
Expected: не меньше 1. Сон без выхода в состояние — генерация текста в никуда.

- [ ] **Step 7: Commit**

```bash
cd ~/hermes-inner-life
git add skills/inner-life/references/dreams.md
git commit -m "feat: night dreams with topic dedup and spark handoff"
```

---

### Task 5: SKILL.md — точка входа

**Files:**
- Create: `skills/inner-life/SKILL.md`
- Delete: `skills/inner-life/references/.gitkeep`

**Interfaces:**
- Consumes: все три файла из `references/` и шаблон из `templates/`.
- Produces: сам скилл. Имя `inner-life` фиксируется здесь и используется в задачах 6–9.

- [ ] **Step 1: Написать frontmatter**

Файл `skills/inner-life/SKILL.md`, начало — дословно:

```yaml
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
```

- [ ] **Step 2: Написать тело по шаблону Hermes**

Обязательные разделы в этом порядке — таков штатный шаблон скиллов Hermes:

**`# Inner Life`**

**`## Overview`** — два абзаца. Первый: у агента накапливается опыт, но между сессиями от него ничего не остаётся, и каждый день начинается одинаково. Второй: скилл даёт три вещи — состояние из фактов с датами, вечернюю запись и ночное свободное мышление, — а влияние на завтра идёт через короткую сводку в нативной памяти.

**`## When to Use`** — маркированный список триггеров: наступил вечерний прогон; наступил ночной прогон; произошло что-то, что стоит занести в состояние; пользователь просит дневник или свёртку.

Затем `**Don't use for:**` — хранение фактов и процедур: для этого у Hermes есть `memory` и `skill_manage`, дублировать их не нужно.

**`## Three modes`** — таблица-диспетчер:

| Mode | When | Read |
|---|---|---|
| Logging | что-то произошло | `references/state.md` |
| Evening | раз в день | `references/journal.md` |
| Night | тихие часы | `references/dreams.md` |

Прямо указать, что подробности лежат в `references/` и читать их надо только для нужного режима — это и есть экономия контекста.

**`## First run`** — скопировать `templates/state.md` в `inner-life/state.md` рабочей директории, если файла ещё нет; создать `inner-life/journal/` и `inner-life/dreams/`.

**`## What this is not`** — короткий абзац: это не система памяти и не оценка настроения в баллах. Чисел здесь нет намеренно: давность события несёт больше смысла, чем выдуманная шкала.

**`## Common Pitfalls`** — нумерованный список:
1. Записать дневник и не обновить сводку в памяти — завтрашний день ничего не заметит.
2. Писать в состояние оценки вместо фактов.
3. Видеть один и тот же сон, пропустив сверку тем.
4. Забить нативную память длинным текстом и упереться в лимит.
5. Писать записи ради регулярности, когда сказать нечего.

**`## Verification Checklist`** — чекбоксы: `inner-life/state.md` существует; сегодняшняя запись на месте либо день осознанно пропущен; сводка в нативной памяти отражает свежие даты; протухшие строки состояния убраны.

- [ ] **Step 3: Удалить заглушку**

```bash
cd ~/hermes-inner-life && rm -f skills/inner-life/references/.gitkeep
```

- [ ] **Step 4: Проверить frontmatter на соответствие формату Hermes**

Run:
```bash
cd ~/hermes-inner-life && python3 - <<'EOF'
import re
src = open('skills/inner-life/SKILL.md').read()
m = re.match(r'^---\n(.*?)\n---\n', src, re.S)
assert m, 'frontmatter not found or not at the very top'
fm = m.group(1)
name = re.search(r'^name:\s*(\S+)', fm, re.M).group(1)
desc = re.search(r'^description:\s*(.+)$', fm, re.M).group(1)
assert re.fullmatch(r'[a-z0-9-]{1,64}', name), f'bad name: {name}'
assert desc.startswith('Use when '), 'description must start with "Use when "'
assert 'version:' in fm and 'license:' in fm and 'tags:' in fm
print('frontmatter OK:', name)
EOF
```
Expected: `frontmatter OK: inner-life`

- [ ] **Step 5: Проверить, что все упомянутые файлы существуют**

Run:
```bash
cd ~/hermes-inner-life/skills/inner-life && for f in $(grep -oE '(references|templates)/[a-z]+\.md' SKILL.md | sort -u); do test -f "$f" && echo "ok  $f" || echo "MISSING $f"; done
```
Expected: четыре строки `ok`, ни одной `MISSING`.

- [ ] **Step 6: Проверить, что в скилле нет скриптов и запрещённых слов**

Run:
```bash
cd ~/hermes-inner-life && find skills -name '*.sh' -o -name '*.py' | grep . ; grep -rniE 'clawhub|openclaw|clawdbot' skills/
```
Expected: оба вывода пустые.

- [ ] **Step 7: Commit**

```bash
cd ~/hermes-inner-life
git add -A skills/
git commit -m "feat: inner-life SKILL.md entry point"
```

---

### Task 6: README, CHANGELOG, категория для Hub

**Files:**
- Create: `README.md`
- Create: `CHANGELOG.md`
- Create: `skills.sh.json`

**Interfaces:**
- Consumes: имя скилла `inner-life` и пути из Task 5.
- Produces: витрина репозитория. Task 8 публикует её как есть.

- [ ] **Step 1: Написать README**

Файл `README.md`. Порядок разделов важен — первый экран решает, поставят звезду или закроют.

Заголовок и подзаголовок:

```markdown
# hermes-inner-life

> Your agent remembers how things have been going.
```

**Схема потока** — первым блоком после подзаголовка:

```
  things happen        →   state.md          dated facts, no scores
                                ↓
  evening              →   journal entry  +  a short summary in memory
                                ↓                      ↓
  weekly               →   rollup            tomorrow's system prompt
                                ↑                      ↓
  night                →   a dream       →   back into state
```

**`## What Hermes already does`** — таблица, снимающая главное возражение опытного пользователя:

| Hermes gives you | inner-life adds |
|---|---|
| facts and procedures in memory | how things have been going lately |
| skills that improve themselves | an evening journal and a weekly rollup |
| scheduling via cron | free thinking at night |

Прямо написать: скилл не заменяет память Hermes и не соревнуется с `background_review` — он занимает пустое место рядом.

**`## Install`** — два пути, tap первым:

```bash
hermes skills tap add DKistenev/hermes-inner-life
hermes skills install DKistenev/hermes-inner-life/inner-life
```

```bash
hermes skills install https://raw.githubusercontent.com/DKistenev/hermes-inner-life/main/skills/inner-life/SKILL.md
```

**`## Schedule`** — три команды cron со ссылкой на `docs/cron-recipes.md`.

**`## What it looks like`** — фрагмент реальной записи дневника и фрагмент сна, по 5–8 строк каждый. Это продающая часть: показать текст, а не описывать его.

**`## No numbers`** — короткий абзац: состояние намеренно без шкал и баллов. Давность события говорит больше, чем выдуманное значение, и её не нужно ни пересчитывать, ни кому-то доверять.

**`## License`** — MIT.

Последней строкой — ровно одна строка про историю:

```markdown
_Previously an OpenClaw skill pack — see [`v1.0-openclaw`](https://github.com/DKistenev/hermes-inner-life/tree/v1.0-openclaw)._
```

- [ ] **Step 2: Написать CHANGELOG**

```markdown
# Changelog

## 1.0.0 — 2026-07-26

Rebuilt from scratch for Hermes Agent. Not an upgrade of the OpenClaw pack —
a different design that keeps the idea.

- One skill instead of six
- State is dated facts, not scores — time carries the decay
- Evening journal absorbs reflection; no separate SELF.md
- Night dreams check the last two weeks to avoid repeating themselves
- Behavior changes through the native memory summary, not through rules
- No scripts, no dependencies
- Dropped: memory and evolve modules — Hermes covers both natively
```

- [ ] **Step 3: Написать skills.sh.json**

```json
{
  "$schema": "https://skills.sh/schemas/skills.sh.schema.json",
  "groupings": [
    { "title": "Agent Inner Life", "skills": ["inner-life"] }
  ]
}
```

- [ ] **Step 4: Проверить валидность JSON**

Run:
```bash
cd ~/hermes-inner-life && python3 -c "import json;d=json.load(open('skills.sh.json'));assert d['groupings'][0]['skills']==['inner-life'];print('skills.sh.json OK')"
```
Expected: `skills.sh.json OK`

- [ ] **Step 5: Проверить, что README упоминает OpenClaw ровно один раз**

Run:
```bash
cd ~/hermes-inner-life && grep -coi 'openclaw' README.md
```
Expected: `1`. Больше — нарушение позиционирования, меньше — потеряна ссылка на историю.

- [ ] **Step 6: Проверить, что пути установки совпадают с реальной структурой**

Run:
```bash
cd ~/hermes-inner-life && test -f skills/inner-life/SKILL.md && grep -q 'skills/inner-life/SKILL.md' README.md && echo "install path OK"
```
Expected: `install path OK`

- [ ] **Step 7: Commit**

```bash
cd ~/hermes-inner-life
git add README.md CHANGELOG.md skills.sh.json
git commit -m "docs: README, changelog, Hub category"
```

---

### Task 7: Документация установки и расписания

**Files:**
- Create: `docs/install.md`
- Create: `docs/cron-recipes.md`

**Interfaces:**
- Consumes: пути и имена режимов из Task 5, разделы дневника и снов из задач 3–4.

- [ ] **Step 1: Написать docs/install.md**

Разделы: установка через tap; установка прямым URL; что появляется после установки (`~/.hermes/skills/inner-life/`); первый запуск — попросить агента инициализировать состояние; как убедиться, что скилл виден (`hermes skills list`); как удалить.

- [ ] **Step 2: Написать docs/cron-recipes.md — вечерний прогон**

```bash
hermes cron create "0 22 * * *" \
  "Run the inner-life evening routine: read inner-life/state.md, write today's journal entry, refresh the state summary in native memory, then prune stale state entries." \
  --name "inner-life evening"
```

- [ ] **Step 3: Написать ночной и недельный прогоны**

```bash
hermes cron create "0 3 * * *" \
  "Run the inner-life night routine: check the last two weeks of dreams, pick a topic that has not come up, think freely about it, and add anything worth keeping to Sparks." \
  --name "inner-life dream"
```

```bash
hermes cron create "0 21 * * 0" \
  "Run the inner-life weekly rollup: fold the last seven journal entries into one weekly note and age out stale state entries." \
  --name "inner-life weekly"
```

- [ ] **Step 4: Добавить раздел о загрузке скилла**

Раздел `## Making sure the skill loads`. Написать: Hermes подтягивает скилл по описанию, поэтому промпт задания содержит слова `inner-life` и название режима — это и есть триггер. Если в установленной версии CLI доступен флаг привязки скиллов к задаче, его стоит добавить; проверить командой:

```bash
hermes cron create --help
```

Явно указать: без такого флага рецепты работают за счёт формулировки промпта, и это рабочий вариант, а не обходной путь.

- [ ] **Step 5: Проверить, что все три расписания различны и корректны**

Run:
```bash
cd ~/hermes-inner-life && grep -oE '"[0-9 */,-]+"' docs/cron-recipes.md | head -3
```
Expected: три разные строки — `"0 22 * * *"`, `"0 3 * * *"`, `"0 21 * * 0"`.

- [ ] **Step 6: Финальная проверка чистоты репозитория**

Run:
```bash
cd ~/hermes-inner-life && echo "--- scripts:"; find skills -name '*.sh' -o -name '*.py' | grep . || echo none
echo "--- forbidden words:"; grep -rniE 'clawhub|clawdbot' skills/ docs/install.md docs/cron-recipes.md README.md || echo none
echo "--- numeric metrics:"; grep -rnE '\b0\.[0-9]\b' skills/ || echo none
```
Expected: `none` во всех трёх проверках.

- [ ] **Step 7: Commit**

```bash
cd ~/hermes-inner-life
git add docs/install.md docs/cron-recipes.md
git commit -m "docs: install guide and cron recipes"
```

---

### Task 8: Публикация репозитория

**Требует явного подтверждения пользователя перед началом.** Задача необратима в части переименования и замены содержимого.

**Files:**
- Modify: удалённый репозиторий `DKistenev/openclaw-inner-life`

**Interfaces:**
- Consumes: готовое содержимое из задач 1–7.

- [ ] **Step 1: Пометить текущее состояние тегом**

```bash
cd /tmp && rm -rf oil-archive && git clone -q https://github.com/DKistenev/openclaw-inner-life.git oil-archive
cd oil-archive && git tag v1.0-openclaw && git push origin v1.0-openclaw
```

Проверка: `gh api repos/DKistenev/openclaw-inner-life/tags --jq '.[].name'` содержит `v1.0-openclaw`.

- [ ] **Step 2: Переименовать репозиторий**

```bash
gh repo rename hermes-inner-life --repo DKistenev/openclaw-inner-life --yes
```

Проверка: `gh repo view DKistenev/hermes-inner-life --json name,stargazerCount` показывает 12 звёзд — переименование их сохраняет.

- [ ] **Step 3: Развести рабочие доки и публичное содержимое**

Русскоязычные спека и план не должны попасть в `main`. Они уезжают на ветку `planning`:

```bash
cd ~/hermes-inner-life
git remote add origin https://github.com/DKistenev/hermes-inner-life.git
git branch -M main
git branch planning
git push origin planning
git rm -r --cached docs/superpowers && printf 'docs/superpowers/\n' >> .gitignore
git add .gitignore && git commit -m "chore: keep planning docs on the planning branch"
```

Проверка перед пушем — в `main` не осталось русского текста:

```bash
cd ~/hermes-inner-life && git ls-files | xargs grep -lP '[\x{0400}-\x{04FF}]' || echo "main is English-only"
```
Expected: `main is English-only`

- [ ] **Step 4: Заменить содержимое main**

```bash
cd ~/hermes-inner-life && git push --force origin main
```

Форс здесь намеренный: старое содержимое сохранено тегом на шаге 1.

- [ ] **Step 5: Заполнить метаданные репозитория**

```bash
gh repo edit DKistenev/hermes-inner-life \
  --description "Your agent remembers how things have been going. Dated state, an evening journal, and dreams at night — for Hermes Agent." \
  --homepage "https://github.com/DKistenev/hermes-inner-life" \
  --add-topic hermes-agent --add-topic ai-agent --add-topic agent-skills \
  --add-topic llm --add-topic ai-memory
```

- [ ] **Step 6: Проверить, что тап резолвится**

Run:
```bash
gh api repos/DKistenev/hermes-inner-life/contents/skills/inner-life --jq '.[].name'
```
Expected: `SKILL.md`, `references`, `templates` — структура совпадает с эталоном `anthropics/skills`.

- [ ] **Step 7: Проверить сохранность звёзд и редиректа**

Run:
```bash
gh api repos/DKistenev/openclaw-inner-life --jq '.full_name, .stargazers_count'
```
Expected: `DKistenev/hermes-inner-life` и `12` — GitHub отдаёт редирект со старого имени.

---

### Task 9: Дистрибуция

**Требует явного подтверждения пользователя по каждому пункту.** Все действия — внешние и публичные.

- [ ] **Step 1: Подать в Hermes Atlas**

Открыть issue в `ksimback/hermes-ecosystem` со ссылкой на репозиторий и двумя-тремя предложениями о том, что это и почему это не дубль нативной памяти. Формат подачи свободный, критерий модерации — Hermes-native.

- [ ] **Step 2: Опубликовать в Skills Hub**

```bash
hermes skills publish skills/inner-life --to github --repo DKistenev/hermes-inner-life
```

Если CLI недоступен локально, пропустить: репозиторий уже работает как tap, и это самостоятельный путь установки.

- [ ] **Step 3: Подготовить черновик ответа Ben'у**

Issue #1 в репозитории, автор `itsgroup-co-nz`, написан 12 апреля по-русски. Подготовить черновик ответа и **отдать пользователю на правку — не отправлять самостоятельно.** Это личная переписка.

Содержание черновика: что проект пересобран под Hermes; что числовую модель эмоций сознательно убрали и почему; ответ на его вопрос о том, как затухание влияет на решения — теперь через сводку в системном промпте, а не через пороги.

- [ ] **Step 4: Анонс в Discord Nous Research**

Короткое сообщение в комьюнити-канал со ссылкой и строкой установки через tap. Текст согласовать с пользователем.

- [ ] **Step 5: Тексты для Show HN и Reddit**

Написать заново, не переносить старые. Угол: агент, у которого состояние — это даты, а не баллы. Публикацию решает пользователь отдельно.

---

## Self-Review

**Покрытие спеки:**

| Требование спеки | Задача |
|---|---|
| Переименование в `hermes-inner-life` + тег `v1.0-openclaw` | 8.1, 8.2 |
| Один скилл, ноль скриптов | 1, 5, проверки 5.6 и 7.6 |
| Состояние без чисел, факты с датами | 2, проверки 2.3 и 7.6 |
| Секции Contact / Friction / Sparks / Waiting + устаревание | 2 |
| Дневник со втянутой рефлексией | 3.2 |
| Сводка в нативную память как канал влияния | 3.3 |
| Недельная свёртка | 3.4 |
| Сны: сверка тем, выход в Sparks, право промолчать | 4 |
| Формат frontmatter Hermes | 5.1, проверка 5.4 |
| Структура тапа `skills/<имя>/SKILL.md` | 1, проверка 8.5 |
| `skills.sh.json` | 6.3 |
| README со схемой потока | 6.1 |
| Два пути установки | 6.1, 7.1 |
| Cron-рецепты | 7.2, 7.3 |
| Topics, homepage, описание | 8.4 |
| Hermes Atlas, Hub, Discord, Show HN | 9 |
| Ответ Ben'у | 9.3 |

Пропусков нет.

**Согласованность имён:** секции состояния `Contact`, `Friction`, `Sparks`, `Waiting` заданы в Task 2 и проверяются на совпадение в задачах 3 и 4. Пути `inner-life/state.md`, `inner-life/journal/YYYY-MM-DD.md`, `inner-life/journal/weekly/YYYY-Www.md`, `inner-life/dreams/YYYY-MM-DD.md` заданы один раз и используются дословно. Имя скилла `inner-life` фиксировано в Task 5 и используется в задачах 6–9.

**Открытый пункт, не блокирующий работу:** наличие флага привязки скиллов к cron-заданию в CLI подтверждено только для Python-API (`cronjob(skills=[...])`). Task 7.4 проверяет это через `hermes cron create --help` и даёт рабочий запасной вариант — триггер через формулировку промпта.
