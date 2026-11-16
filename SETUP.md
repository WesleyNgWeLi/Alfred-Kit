# SETUP.md -- instructions for Claude

You are setting up a personal work assistant (default name "Alfred") for the person in this chat, called "the owner" below. You will interview them, write their configuration, generate their files from `templates/`, publish their own private page, and schedule their routines. Nothing here belongs to anyone else: this kit ships with no data, and you must not copy data from any other person's assistant.

Modes. Pick from what the owner said:
- **Setup** (default, or "set me up"): run Steps 0-9.
- **Reconfigure** ("reconfigure", "change my schedule", "add a folder to ignore"): see "Reconfigure" at the end.
- **Upgrade** ("upgrade", "I got a newer kit"): see "Upgrade" at the end.
- **Uninstall** ("remove it", "uninstall"): see "Uninstall" at the end.

The kit root is the folder that contains this file. Call it `<KIT>` and resolve it to an absolute path now. Every generated file lives inside `<KIT>`.

## Ground rules for this setup (HARD)

- Never send, draft, forward, or reply to mail or Teams messages. Never create or change Outlook rules, folders, filters, categories, or calendar events. Never launch or automate an Outlook client (classic, new, or on the web): a dormant client can send stale Outbox mail when it opens.
- The only things you create or change: files inside `<KIT>`; `~/.claude/skills/<command>/SKILL.md`; scheduled tasks whose id starts with the owner's task prefix; and the one new artifact you publish in Step 4 plus its database.
- Ask before overwriting anything that already exists (a skill folder, a scheduled task id, a `config.md`).
- Mail and Teams reads during setup are read-only and only for the optional discovery assist and report lookup in Step 1. Never store message bodies in config.
- The page stays private. Do not change its sharing. If the owner wants to share it, tell them it holds their personal work data and they should use the page's own Share menu only if they really mean to.
- Show the owner what you write: the config summary before saving, and a list of every file, task and artifact created at the end.

## Step 0 -- Preflight

Check, and report in one short list:
1. **Claude desktop app, Code tab.** Scheduled routines run only in the desktop app, and only while it is open. If this session is not the desktop app, say so: setup can continue, but routines will not run until they set it up there.
2. **Microsoft 365 connector** (tools named like `outlook_email_search`, `outlook_calendar_search`, `chat_message_search`, `read_resource`). Load them with ToolSearch if they are deferred. If missing: tell the owner to enable the Microsoft 365 connector (claude.ai Settings -> Connectors, then allow it in Claude Code) and stop until it is available. Without it the assistant has nothing to read.
3. **Artifacts with a database**: the `Artifact` and `ArtifactData` tools. If missing, stop and say the account needs Artifacts enabled.
4. **Scheduled tasks**: `create_scheduled_task`, `list_scheduled_tasks`, `update_scheduled_task`, `run_scheduled_task`, `delete_scheduled_task` (load with ToolSearch). If missing, continue but note routines must be added later.
5. **Timezone**: note this machine's timezone. Routines run on this machine's clock.
6. If `<KIT>/config.md` already exists, this is not a fresh setup: ask whether to reconfigure instead.

Then explain in 3-4 lines what they are getting (a private page with Queues, Today, Digest, an afternoon check-in, a weekly review, an optional weekly status-report draft, and a Portfolio view; a morning sweep of their mail, Teams and calendar; a `/command` that shows what to do next, opens any item with its full thread, and re-runs a missed routine; read-only, never sends anything) and that setup takes about 10-15 minutes of questions.

## Step 1 -- Interview

Ask in rounds of at most 4 questions. Use `AskUserQuestion` for choices when it is available, plain chat for free text. Always offer the default so "use the default" or "skip" works. Do not ask what you can derive (e.g. timezone from the system, the command from the name). Keep a running draft of answers in the shape of `templates/config.template.md`.

**Round A -- Basics**
- What should the assistant be called? Default "Alfred". Offer 3 alternatives if they want ideas (a sidekick name works well: it preps everything and never acts on its own).
- Their full name, the short name briefs should use, their role/team, and their manager's name (optional).
- Their work email (used only to find their own sent mail and past reports).
- Confirm the detected timezone and workdays (default Mon-Fri). Record `routines.machine_timezone`. Times are always asked and stored in the owner's timezone; if it differs from the machine's, tell them Step 8 converts the schedule to the machine's clock.
- Derive `command` (lowercase name, letters/digits/dashes) and `task_prefix` (same). Tell them the command, e.g. `/alfred <id>`. If `~/.claude/skills/<command>` already exists or the name collides with a built-in command (`/loop`, `/review`, `/init`, `/help`, `/config`, `/schedule`, `/run`), propose another.

**Round B -- Routines and times**
- Morning sweep time (default 07:30; it is always on, because it keeps the data current). Open the brief with a joke? (default yes)
- Afternoon check-in: on or off, time (default 16:00), and its label (default "4 PM check-in", matching the time they chose).
- Weekly review: on or off, day and time (default Friday 15:00), and its label (default "Weekly review").
- How far back the first sweep should read (default 7 days; more means a slower first run).

**Round C -- Weekly status report** (skip if the weekly review is off)
- Do they send a regular status update (to a manager, a team, a client)? If not, set `weekly_report.enabled: false`.
- If yes: what is it called (e.g. "Weekly update", "Friday notes"), who reads it, the title format, whether the title's dates name the week just finished or the coming week (`title_period`: `completed` | `upcoming`), and its sections. Offer three ways to get the sections: use the default four (Priorities, Accomplishments, Challenges, Information); they paste or describe their format; or, with their permission, you find their last one in sent mail and read its headings (read-only). Give each section a `role` (priorities, accomplishments, blockers, info, other); exactly one is `accomplishments`, and if none fits, ask which section should hold finished work. Ask whether any section is one they always write themselves (`self_written: true`; it is left empty and suggestions go to a "For you" list). Record the search that finds their sent reports as `previous_report_search` (`outlook_email_search` arguments: a `query` and the `recipient` it goes to, no `folderName`).
- Tell them how drafting works: each week the assistant edits their last sent one for the new week (carry forward, update what moved, move finished work to accomplishments) and lists every change in a "For you" section, so the draft reads like theirs. It only drafts; they always send it themselves.

**Round D -- What they work on**
- Their "Drive" initiatives: the 1-5 things they own and must not let slip. For each: a short name and the signals that identify its mail and chats (people, project names, vendors, keywords). Tell them they can move initiatives between Drive and FYI, and archive finished ones, on the page later; this list only seeds it.
- Their "FYI" buckets: things they follow but do not own (default one bucket, "Admin and HR"). The assistant surfaces only direct asks from these.
- May the assistant create new FYI buckets on its own when a recurring topic appears? (default yes, announced in the brief)

**Round E -- Mail and Teams**
- Folders to ignore completely (e.g. Newsletters, automated alerts). For each, the sender or subject pattern that identifies its mail (its `signature`, e.g. "sender alerts@vendor.com, or [ALERT] in the subject"), because sweeps drop this mail by pattern rather than looking up each folder on every run. If they do not know, the discovery assist below can propose one from the folder's recent senders; `none` is allowed for a catch-all folder with no pattern.
- Folders to summarize but never turn into tasks (e.g. a Jira or ServiceNow notifications folder): for each, the hours to look back, which run shows it (default morning), a label like "Jira, last 24h", and its `signature` (sender or subject pattern, as for ignored folders). The first summary folder's label also becomes the page's `summaryLabel`.
- VIP senders: people whose mail must never be filtered and is always treated as important (manager, key stakeholders). Store each as "Name <address>"; if they give only names, look the addresses up with `search_people` (read-only) and confirm them.
- Senders and kinds of mail to always skip (automated notifications, marketing, specific distribution lists). Personal and family mail is ignored by default.
- People whose meeting-recap emails assign them tasks (e.g. a project manager who sends notes with owners). Same "Name <address>" format.
- Include Teams chats? (default yes) Check Drafts and Outbox for unsent replies? (default yes)

**Optional discovery assist.** After Round E, offer: "Want me to look at the last 14 days of your mail (subjects, senders and folders only, read-only) and suggest folders to ignore, noise senders, VIPs, and initiative signals?" Only with a yes: run `outlook_email_search` with `query: "*"`, `afterDateTime` = 14 days ago, no folder, a few pages; group by sender and folder; propose lists; let them accept, edit or reject each. For each folder they chose to ignore or summarize, also run one `outlook_email_search` with that `folderName` over the same 14 days and propose a `signature` from its senders and subject prefixes. Do not read bodies. Do not save anything they did not accept.

**Round F -- Guardrails and style**
- Any confidentiality rules: clients or programs whose details must never be mixed, topics never to summarize. Each becomes a hard rule.
- Style: no em dashes (default on), and anything else ("short bullets", "plain language"). Claims are always labeled VERIFIED/INFERRED/UNKNOWN; that is not optional.

## Step 2 -- Confirm and write config

Show a compact summary of every answer (one line per setting, grouped like the config), and ask "Save this?". Apply edits until they say yes. Then write `<KIT>/config.md` from `templates/config.template.md` with every `<...>` filled or removed and `artifact_url` left as `pending`.

## Step 3 -- Generate the page

Copy `templates/page.template.html` to `<KIT>/artifact/<command>.html` and change only:
- `<title>Alfred</title>` and `<h1 id="app-name">Alfred</h1>`: the assistant name.
- The JSON between `/*CONFIG*/` and `/*END CONFIG*/`, exactly this shape (valid JSON, double quotes):
  ```json
  {"name":"<name>","command":"/<command>",
   "morning":{"when":"<e.g. 7:30 AM>"},
   "afternoon":{"enabled":<true|false>,"label":"<label>","when":"<e.g. 4 PM>"},
   "weekly":{"enabled":<true|false>,"label":"<label>","when":"<e.g. Friday 3 PM>"},
   "report":{"enabled":<true|false>,"name":"<report name>"},
   "summaryLabel":"<first summary folder label, or Summaries>"}
  ```
  `when` values are clock times in the owner's timezone, in the form "8 AM" or "4:30 PM" (minutes only when not :00); the weekly one is prefixed with the day, e.g. "Thursday 2 PM". For a routine that is off, set `enabled` to false and keep the default label and `when`; they are not shown.
Change nothing else in the page. Its static tab labels and `<!-- SECTION -->` comments are expected; the page sets labels from the config block at runtime.

## Step 4 -- Publish the page

1. If an `artifact-capabilities` skill is listed, load it first (it governs the `capabilities` argument).
2. Publish with the `Artifact` tool: `file_path` = `<KIT>/artifact/<command>.html`, `title` = the assistant name, `icon` = `checklist`, `description` = "<Name>: what I am driving, what I owe, and who I am waiting on.", and `capabilities` = `{"db":{"rules":[{"path":"","read":"view","write":"owner"}]}}` (anyone the owner shares with may view; only the owner writes).
3. Record the returned URL as `assistant.artifact_url` in `config.md`.
4. Tell the owner the page is private, and offer to pin it to their claude.ai sidebar (`pin: true` only on a yes).

## Step 5 -- Seed the database

One `ArtifactData` `batch` on the new URL:
- One `initiatives/<id>` per Drive and FYI initiative: `{name, tier, order (Drive first, 1..n), status: "green", status_why: "Set by the first sweep", outcome: "", next_action: "Set by the first sweep", last_touch: null, links: [], stakeholders: [], archived: false}`.
- `meta/sweep`: `{last_sweep_at: <now minus first_sweep_lookback_days, ISO UTC>, last_sweep_kind: "Setup", filter_version: 1}`.
Read every doc back and confirm.

## Step 6 -- Generate the rules, routines and command

Fill tokens from `config.md`:

| Token | Value |
|---|---|
| `{{NAME}}` | assistant.name |
| `{{COMMAND}}` | assistant.command |
| `{{TASK_PREFIX}}` | assistant.task_prefix |
| `{{ARTIFACT_URL}}` | assistant.artifact_url |
| `{{OWNER_FULL}}`, `{{OWNER_SHORT}}`, `{{OWNER_ROLE}}`, `{{OWNER_EMAIL}}` | owner.* |
| `{{AFTERNOON_LABEL}}`, `{{WEEKLY_LABEL}}` | routines.*.label |
| `{{REPORT_NAME}}`, `{{REPORT_TITLE_FORMAT}}`, `{{REPORT_AUDIENCE}}` | weekly_report.* |
| `{{PREVIOUS_REPORT_SEARCH}}` | weekly_report.previous_report_search as inline code arguments, e.g. `` `query: "Status"`, `recipient: "dana@example.com"` `` (only used inside text that is kept when the search is set) |
| `{{OWNER_TIMEZONE}}` | owner.timezone |
| `{{MORNING_TIME}}` | routines.morning.time, as a clock time like "7:30 AM" |
| `{{KIT_PATH}}` | `<KIT>` |

When a feature is off, `{{REPORT_NAME}}`, `{{AFTERNOON_LABEL}}` and similar tokens for it are not needed: the text that uses them must be removed (see the checklist below).

Write:
- `<KIT>/spec/sweep-spec.md` from `templates/spec.template.md`.
- `<KIT>/tasks/morning.md`; `<KIT>/tasks/afternoon.md` if the afternoon check-in is on; `<KIT>/tasks/weekly.md` if the weekly review is on.
- `<KIT>/skills/<command>/SKILL.md` from `templates/skill-item.template.md`.

Template syntax. Only these are setup-time; resolve all of them:
- `{{TOKEN}}`: replace from the table above.
- `[[GENERATE if <condition>]] ... [[/GENERATE]]`: keep the lines between (and resolve fills inside) when the condition is true; delete the whole block, markers included, when it is false. `[[GENERATE from ...: <how>]]` blocks are always kept and filled as described.
- `[[EACH <thing>: <format>]]`: one entry per config item in that format, or a comma-separated run when the marker says so.
- `[[IF <condition>: text]]` and `[[IF <condition>: a | ELSE: b]]`: keep `text` (or `a`) when true, nothing (or `b`) when false. Shorthand conditions: `report` = `weekly_report.enabled` (and only when the weekly review is on), `afternoon` = `routines.afternoon.enabled`, `weekly` = `routines.weekly.enabled`, `teams` = `teams.enabled`. Any other condition naming a config value (`previous_report_search`, `style.extra`, `excluded_folders`) is true when that value is set and non-empty. Text after `IF <condition>:` starts with exactly one space, or none when it begins with punctuation; tidy doubled spaces after filling.
- `[[GENERATE ...]]` blocks may nest; resolve the inner one with the same rule.
- `[[folder]]`, `[[signature]]`, `[[label]]` and similar inside a per-item block: that item's config value.
- `<!-- ... -->` comments in these templates are setup notes: delete them.
Everything else in `<angle brackets>` (e.g. `<id>`, `<today>`, `<YYYY-MM-DD>`, `<now ISO UTC>`) is runtime notation the routines read: leave it exactly as written.
**Feature-off checklist.** Markers do not cover every mention. After filling, reread each generated file and remove every rule, section, field, list entry and phrase that applies only to a feature that is off, keeping the rest of each sentence grammatical:
- Afternoon check-in off: the spec's "Afternoon check-in" section, its task id, the `pm` report id and kind, the afternoon calendar rule, `-pm` retention, "the afternoon check-in does not write ..." clauses, and the command's `sweep pm`.
- Weekly review off: the weekly task id, `wk` digests, the `review` and `status` reports, the Retention section (move retention into the morning task on Mondays instead), the "Noise check" sentence in "Noise filter" (the `[[IF weekly:]]` handles it), weekly-run clauses, and the command's `sweep weekly`.
- Weekly report off: the spec's "{{REPORT_NAME}} drafting" section, the brief's report check (0b), and every `{{REPORT_NAME}}`, `status` report, `status-reports/`, accomplishments-for-the-report and "Deprioritized" mention; say once in Reports that there is no weekly report.
- Teams off: Teams in sources, extraction rules, sources[] names, task descriptions ("email + Teams"), and the item command's `teams` branch.
- No VIPs, summary folders, excluded folders, noise senders, recap senders or confidentiality rules: their bullets (including an empty "Senders:" line), sources[] names ("VIP check", "each summary folder") and brief sections.
When done, search the generated `spec/`, `tasks/` and `skills/` files for `{{`, `[[` and `<!--` and fix anything left. If `weekly_report.enabled` is false, the generated spec and weekly task must not mention a status report except to say there is none. Keep the quoting on each `description:` line (it contains a colon).

## Step 7 -- Install the item command

Write `~/.claude/skills/<command>/SKILL.md` (ask first if it exists) with exactly the following, copying the `description:` line verbatim, quotes included:

```markdown
---
name: <command>
description: <copied verbatim from <KIT>/skills/<command>/SKILL.md>
---

Read `<KIT>/skills/<command>/SKILL.md` and follow it exactly, passing along the argument (none, an item id, "next", or "sweep" with its optional routine). That file is the canonical version of this command; this copy only makes it available in every Claude Code session.

If that file cannot be read, say the <Name> folder is unavailable and stop. Guardrails regardless: never send, draft, forward, or reply in Outlook or Teams; never launch any Outlook client; drafts go in chat only.
```

Use the absolute `<KIT>` path. On Windows write it as UTF-8 without a BOM.

## Step 8 -- Schedule the routines

`list_scheduled_tasks` first; if an id below exists, ask before replacing it. Then `create_scheduled_task` for each enabled routine with `notifyOnCompletion: false`:

| taskId | title | cronExpression (local time) | tasks file |
|---|---|---|---|
| `<prefix>-morning` | `<Name> morning brief` | `<mm> <hh> * * <days>` | `tasks/morning.md` |
| `<prefix>-afternoon` | `<Name> <afternoon label>` | `<mm> <hh> * * <days>` | `tasks/afternoon.md` |
| `<prefix>-weekly` | `<Name> <weekly label>` | `<mm> <hh> * * <day>` | `tasks/weekly.md` |

Days: Mon=1 ... Fri=5, Sat=6, Sun=0; "Mon-Fri" is `1-5`. Cron runs on this machine's clock: if the owner chose times in another timezone, convert them and say so. `description` = the text of the `description:` line of that tasks file, without the quotes. `prompt`, exactly:

```
Read <KIT>/tasks/<file> and follow it exactly. That file is the canonical prompt for this run; the rules it references are in <KIT>/spec/sweep-spec.md.

If that file cannot be read, stop and report that the <Name> folder is unavailable. Do not improvise a sweep. HARD guardrails regardless: read-only on Outlook and Teams; never send, draft, forward, reply, label, delete, accept, or post; never launch or automate any Outlook client.
```

Then `list_scheduled_tasks` again and confirm each task's schedule and next run.

## Step 9 -- First run and hand-off

1. Offer to run the morning sweep now (`run_scheduled_task` on `<prefix>-morning`) so the page fills in. Warn that the first run reads the whole look-back window and can take several minutes. If they decline, the first scheduled run does it.
2. Finish with a short summary:
   - The page link, and the tabs they will see.
   - Each routine with its schedule. Routines run only while the Claude desktop app is open; a missed run catches up on the next one.
   - How to use it: Done and Drop on the page; "Drive this" / "Move to FYI" and Archive to reshape their initiatives (sweeps never override those). The Claude button copies `/<command> <id>`, which they paste into a new Claude Code session to get the thread and a draft reply. `/<command>` alone shows what to do next; `/<command> sweep` re-runs a missed morning run (the app was closed, a connector was off). The weekly draft is in `<KIT>/status-reports/` and on its tab, with a Copy button that keeps the bullets; they send it themselves.
   - How to change anything: open Claude Code in `<KIT>` and say "Read SETUP.md and reconfigure".
   - A list of every file, task and artifact you created.
3. Remind them not to share their configured folder. To share the assistant with someone else, they share the original kit, not their copy (their copy holds their config and drafts).

## Reconfigure

1. Read `<KIT>/config.md`. Ask what they want to change (or walk the interview rounds they pick).
2. Show the before/after of each changed setting and confirm.
3. Regenerate only what the change affects:
   - Name, command, labels, routines on/off, report name: the page (Step 3), then republish to the same `artifact_url` (read the live artifact with `Artifact` `read` first and build on it; keep the URL). A command rename also moves `~/.claude/skills/<old>` to the new name (ask first).
   - Times or days: `update_scheduled_task` on the existing ids (`cronExpression` only).
   - A routine turned on or off: create, or ask and then delete, that task; regenerate its tasks file and the page.
   - Anything else (initiatives, folders, VIPs, noise, style, report sections): regenerate `spec/sweep-spec.md` and the affected tasks files. New initiatives also get seeded in the database; removed ones are kept in the database (their items keep history) unless the owner asks to delete them. Never overwrite `tier` or `archived` on an existing initiative: the owner sets those on the page. If `config.md` and the database disagree on a tier, say so and point them to the page buttons.
4. List every file, task and database doc changed.

## Upgrade

For an owner who set up from an older kit and now has a newer one in the same folder (they copied the new `README.md`, `SETUP.md` and `templates/` over the old ones; their `config.md` and generated folders are untouched).
1. Read `<KIT>/config.md` and compare it with `templates/config.template.md`. For each setting the new template adds, ask with its default (for example: a `signature` for each excluded folder, run the discovery assist's folder step if they want help; the report's `title_period` and any `self_written` sections). Show the before/after and save.
2. Regenerate everything from the new templates with the saved config: `spec/`, `tasks/`, `skills/<command>/SKILL.md` (Step 6), the page (Step 3), then republish the page to the same `artifact_url` (read the live artifact with `Artifact` `read` first; keep the URL). Rewrite the `~/.claude/skills/<command>/SKILL.md` pointer from Step 7 so its `description:` matches.
3. Do not migrate the database. New fields are optional: the next sweep writes `status_why` and the new status colors, and closes made before the upgrade simply have no `closed_by`. Never reset `tier` or `archived`.
4. Leave scheduled tasks alone unless a tasks file name changed. List every file and doc changed, and offer `/<command> sweep` to see the new rules work.

## Uninstall

Confirm each step before doing it:
1. Delete the scheduled tasks with the owner's prefix.
2. Delete `~/.claude/skills/<command>/`.
3. Delete the artifact (`Artifact` `delete` on `artifact_url`). This permanently removes the page and all its data for everyone; say so plainly and get an explicit yes.
4. Tell them they can delete `<KIT>` themselves; do not delete it for them.
