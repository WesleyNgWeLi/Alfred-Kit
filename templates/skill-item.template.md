---
name: {{COMMAND}}
description: 'Front door to {{NAME}}. "/{{COMMAND}}" shows what to do next, "/{{COMMAND}} <item-id>" opens one item with its full context (initiative, related open items, source email or Teams thread) and helps {{OWNER_SHORT}} close it, and "/{{COMMAND}} sweep" runs a manual catch-up of a missed routine. Use when {{OWNER_SHORT}} says "{{COMMAND}}", pastes a {{NAME}} item id, asks what is next on their list, or asks to re-run a missed sweep.'
---

# {{NAME}}

The rules live in `spec/sweep-spec.md` at the kit root, two folders up from this file (on this machine `{{KIT_PATH}}/spec/sweep-spec.md`). Read its "Data" and "Guardrails" sections before any write.

Database: {{ARTIFACT_URL}}. Load `ArtifactData` with ToolSearch `select:ArtifactData`.

## Pick the mode from the argument

| Argument | Mode |
|---|---|
| none, `next`, or an id that does not resolve | **Next** (read-only) |
| an item id (`m-...`, `tm-...`, `d-...`, or a random manual id) | **Item** |
| `sweep`[[IF afternoon: , `sweep pm`]][[IF weekly: , `sweep weekly`]] | **Sweep** |

Anything else (for example "what is due today"): answer it read-only from {{NAME}}'s data using the Next mode's reads, and say which mode would act on it.

## Next (read-only)

1. Read in parallel: `meta/sweep`, `meta/recommendations`, `initiatives` (skip `archived: true`), and open `items` (`query` with `state == "open"`).
2. **Freshness line first.** "Last sweep: <local time>, <last_sweep_kind>." Find the most recent workday whose {{MORNING_TIME}} (local) is more than an hour past. If `last_sweep_at` is before {{MORNING_TIME}} on that day, the morning run did not happen: say so on the same line and suggest `/{{COMMAND}} sweep`.
3. **Top 5**, in this order, oldest first within each group, never an item that is `done` or `dropped`:
   1. Open items past `due`.
   2. `ask_to_me` on a Drive initiative.
   3. `my_commitment` due by the end of the next workday.
   4. Any other `ask_to_me` 2+ days old.
   5. Everything else by age.

   One line each: `id`, type, text, counterpart, age in days, due if any, link.
4. **Stored recommendations**: show `queues` and `today` from `meta/recommendations` only if still live (the item is open, the meeting has not ended).
5. End with: "Work one: `/{{COMMAND}} <id>`." Write nothing in this mode.

## Item

### 1. Load context (read-only, in parallel where possible)

1. `get` `items/<id>`. Keep its `version`.
2. `get` `initiatives/<initiative_id>` (name, tier, status, status_why, next_action, decision_needed, risk, pm_move).
3. `query` `items` where `initiative_id == <initiative_id>`; keep the open ones. Also flag other open items with the same `counterpart`.
4. Source thread, by `source`:
   - `mail`: take the `ItemID` from `link`, URL-decode it, replace `/` with `-` and `+` with `_`, then `read_resource` `mail:///messages/<id>`. If that fails, `outlook_email_search` with `sender` = counterpart and `afterDateTime` = one day before `created`, pick the message whose received time matches `created`, and read it. Then look for {{OWNER_SHORT}}'s later replies in the thread (`recipient` = the counterpart's address, no folder).
   - `teams`: `chat_message_search` with `sender` = counterpart around `created`; read the thread with `read_resource`.
   - `manual` or no link: work from the item text and say the source is unavailable.
   - A connection failure means the thread is unavailable, not empty. Say so.
5. If the item comes from a meeting series or recap, check `meetings` for the next occurrence.

### 2. Brief {{OWNER_SHORT}} (short, then stop and wait)

- **The ask:** one line: who wants what, how old (days since `created`), due date if any.
- **The thread:** 2-4 bullets on what was actually said, and what {{OWNER_SHORT}} already replied.
- **Related:** other open items on this initiative or with this person that one reply could cover.
- **Recommended next step:** one verb-first action. If it is a reply, give a ready-to-paste draft in chat.

Label claims VERIFIED (read this session) / INFERRED / UNKNOWN. Follow the spec's style and confidentiality rules.

### 3. Close out (only when {{OWNER_SHORT}} says so)

- State change: `update` `items/<id>` with `if_version` from step 1: `{state: "done" | "dropped", closed_at: <now ISO UTC>, closed_by: "skill"}`. `done` only when {{OWNER_SHORT}} did it; `dropped` when they chose not to. Show the before/after of every field written, then read it back.
- Other changes (due date, text, initiative `next_action`) follow the same pin-and-read-back rule.
- Never reopen a closed item, never delete items, never touch `meta/*`, never write `tier` or `archived` on an initiative (the page owns them).

## Sweep (manual catch-up)

Use when a scheduled run was missed or aborted (the desktop app was closed, a connector was missing), or {{OWNER_SHORT}} asks for a run now.

1. **Pick the task.** `sweep` = `tasks/morning.md`.[[IF afternoon:  `sweep pm` = `tasks/afternoon.md` ({{AFTERNOON_LABEL}}).]][[IF weekly:  `sweep weekly` = `tasks/weekly.md` ({{WEEKLY_LABEL}}).]]
2. **Preflight, before any read that leads to a write:**
   - Connectors: ToolSearch for `outlook_email_search` and `outlook_calendar_search`[[IF teams: , plus `chat_message_search`]]. If any is missing, stop: name the missing connector, and say nothing was written and `last_sweep_at` was not advanced. Never run a partial sweep.
   - Duplicate check: for `sweep`[[IF weekly:  or `sweep weekly`]], read `meta/sweep`; if a run of the same kind already finished today (local), say when and ask before re-running.[[IF afternoon:  For `sweep pm`, `get` `reports/<today>-pm`; if it exists, say when it was generated and ask.]] A re-run is safe (it sweeps from `last_sweep_at` and replaces the same-day digest or report) but wasteful.
3. **Run.** Read the chosen task file from the kit folder and follow it exactly, as a one-off manual run, not the scheduled task. Its outputs, guardrails and chat summary are unchanged.

## Hard guardrails (every mode)

- Never send, draft, forward, or reply in Outlook or Teams. No `outlook_create_draft`, `outlook_send_*`, `outlook_create_reply*`, `teams_send_*`, or any mail or Teams delete or label tool. Drafts go in chat only; {{OWNER_SHORT}} sends them.
- Never launch or automate any Outlook client (classic, new, or on the web).
- If the item is an unsent draft or stale Outbox message, say what to do in Outlook; do not do it.
