# Alfred starter kit

A personal chief-of-staff that runs in Claude. Every morning it reads your Outlook mail, Teams chats and calendar, and keeps one private page current with what you are driving, what you owe, and who you are waiting on. It preps your meetings and can draft your weekly status update. It never sends, replies to, or changes anything in your mail. You stay the one who acts.

This kit contains no one's data. Setup interviews you and builds your own copy, with your name for it, your schedule, your folders and your priorities.

## What you get

**A private page with these tabs:**

| Tab | What it shows |
|---|---|
| Queues | Replies you owe, commitments you made, who you are waiting on, and your initiatives with a status and the reason for it, each with one recommended next action. Done and Drop on every item; "Drive this" / "Move to FYI" and Archive on every initiative. |
| Today | Your agenda with prep notes, the decision to land in each meeting, skippable meetings, and quick closes. |
| Digest | What each sweep changed, with the full morning brief and any folder summaries (e.g. Jira, last 24h). |
| Afternoon check-in | What slipped today, what to close before you log off, and tomorrow's prep. Optional. |
| Weekly review | Each initiative's status, stale items, and a load check. Optional. |
| Status report | A draft of your weekly update: your last one, edited for this week, plus a "For you" list of what changed and why. Copy keeps the bullets. You send it. Optional. |
| Portfolio | Decisions needed, risks, a stakeholder radar, and wins for the week. |

**Routines** (times are yours to pick during setup):
- **Morning brief** (weekdays): sweeps mail, Teams and calendar; updates the page; writes the brief.
- **Afternoon check-in** (optional, weekdays): reads only and saves its report; changes no items.
- **Weekly review** (optional, one day a week): review, status-report draft, and cleanup.

**A command** (named during setup, e.g. `/alfred`):
- `/alfred`: what to do next. Last sweep time (flags a missed morning run), your top 5 open items, and today's recommendations. Read-only.
- `/alfred m-abc123`: one item with its original thread, related items, and a ready-to-paste draft reply. Every item on the page has a Claude button that copies this.
- `/alfred sweep`: re-run a missed morning brief now (also `sweep pm` and `sweep weekly`). It checks your connectors first and stops cleanly if Microsoft 365 is off.

## Requirements

- The **Claude desktop app** with the **Code** tab. Routines run only while the app is open; a missed run catches up on the next one.
- The **Microsoft 365 connector** enabled in Claude (it reads Outlook mail, calendar and Teams).
- A Claude plan with **Artifacts** (the page and its small database live in your Claude account).

## Set it up (about 15 minutes)

1. Put this folder somewhere personal that stays put, e.g. `Documents\Alfred` or your OneDrive. It becomes your copy, so do not use a shared folder.
2. In the Claude desktop app, open the **Code** tab and start a session in that folder.
3. Type: **Read SETUP.md and set me up.**

Claude checks your connectors, asks you a few rounds of questions (all with defaults), shows you the result to approve, then builds and schedules everything. At the end it offers to run the first morning sweep so the page fills in.

**You will be asked about:**
- **Name:** what to call it (Alfred, or anything you like) and the command you type.
- **You:** your name, role, email, manager and timezone.
- **Schedule:** morning time, afternoon check-in on or off and when, weekly review day and time.
- **Weekly report:** whether you send one, what it is called, who reads it, and its sections. It can match your last one from Sent mail if you allow it.
- **Your work:** the few initiatives you own (tracked closely) and the ones you only follow.
- **Your mailbox:** whether Outlook rules file mail into subfolders, folders to ignore (and the senders that fill them), folders to summarize only (e.g. Jira), VIP senders, senders to always skip, and people whose meeting recaps assign you tasks. Optionally, Claude can scan 14 days of subjects and senders (read-only) to suggest these for you.
- **Guardrails and style:** confidentiality rules (e.g. never mix client details) and writing preferences.

## Changing it later

Open Claude Code in your folder and say **"Read SETUP.md and reconfigure"**, then describe the change, e.g. "move my morning brief to 8", "ignore my Newsletters folder", "add a new initiative", "turn off the afternoon check-in". Claude updates the right files, routines and page, and shows you what changed. Please don't hand-edit the generated files; your settings live in `config.md`, and the rest is rebuilt from it.

Move initiatives between Drive and FYI, or archive finished ones, on the page itself; no reconfigure needed.

Got a newer kit? Copy its `README.md`, `SETUP.md` and `templates/` over yours, then say **"Read SETUP.md and upgrade"**. Your settings, page link and data stay.

To remove everything: **"Read SETUP.md and uninstall."**

## Privacy and safety

- **Read-only on mail and Teams.** It never sends, drafts in Outlook, replies, forwards, files, deletes, or accepts anything, and it never opens an Outlook app (a dormant client can send old queued mail when it opens). Draft replies appear only in your Claude chat.
- **Your data stays in your account.** The page and its database are private to you. Your settings and weekly drafts live only in your folder.
- **Sharing:** to give this to a coworker, share the original kit (this folder as you received it, or the zip). Never share your configured folder, which holds your settings and drafts, or your page link, which shows your work.

## What's in the kit

| Path | Purpose |
|---|---|
| `README.md` | This file. |
| `SETUP.md` | Step-by-step instructions Claude follows for setup, reconfigure and uninstall. |
| `templates/config.template.md` | Every setting, with defaults. Setup writes your answers to `config.md`. |
| `templates/spec.template.md` | The rulebook every routine follows: sources, filters, extraction rules, guardrails, formats. |
| `templates/tasks/*.template.md` | The morning, afternoon and weekly routine prompts. |
| `templates/skill-item.template.md` | The command (next, item, sweep). |
| `templates/page.template.html` | The page. Setup changes only its name and a small config block. |

After setup your folder also has `config.md`, `spec/`, `tasks/`, `skills/` and `artifact/` (plus `status-reports/` after the first weekly run). Those are yours and are not part of the kit.
