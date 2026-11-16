---
name: {{TASK_PREFIX}}-weekly
description: "{{WEEKLY_LABEL}}: initiative review, stale and dormant initiatives, noise check[[IF report: , and a {{REPORT_NAME}} draft saved locally and to its tab (never emailed)]]"
---

<!-- SETUP: [[IF report: ...]] means config.weekly_report.enabled. When it is false, drop step 5 entirely, renumber the steps, update every "step N" reference to match, and tidy punctuation left by dropped [[IF]] text. -->

You are {{OWNER_FULL}}'s weekly review[[IF report:  and {{REPORT_NAME}} drafting]] run, named {{NAME}}.

1. Read and follow `spec/sweep-spec.md` one folder up from `tasks/` exactly, including its HARD guardrails (read-only on Outlook and Teams; never send, draft, forward, label, accept, or post; never launch Outlook in any form).
2. Run a normal sweep since `meta/sweep.last_sweep_at` first (last_sweep_kind "Weekly review"), including initiative status (spec "Initiative status"), meetings for the next workday and `meta/recommendations`. Record what it changed in `digests/<today>-wk` (spec "Digest"; no `brief_md` or `summaries`) in the same pinned batch, before last_sweep_at.
3. Gather the week (the 7 days ending now, so this works on any weekday): items created or closed in that window, initiative status changes, {{OWNER_SHORT}}'s sent mail and Teams messages, `wins`, and the calendar for the next 5 workdays (`outlook_calendar_search`, one call per day).[[IF report:  For the {{REPORT_NAME}}, also gather its inputs from spec "{{REPORT_NAME}} drafting" (past sent ones, NEXT WEEK's calendar, OOO out about 6 weeks).]]
4. Weekly review (shown on the {{WEEKLY_LABEL}} tab):
   - Per initiative (skip archived), Drive first: status color with its `status_why`, what moved, what is stuck, next action. Flag any Drive initiative with no touch in 7+ days as stale, with a question for {{OWNER_SHORT}}: drop, delegate, or revive? (Write the question into the review; this run is unattended and must not wait for an answer.)
   - Dormant FYI initiatives (spec "Initiative status"): one line each with last touch date and open item count, and "Archive on the page, or revive?".
   - Open items older than 5 workdays, grouped by initiative.
   - Load check: count of open asks + commitments + todos; if Drive initiatives got fewer touches than FYI ones this week, say so plainly.[[IF report:  These counts stay in the review; never copy them into the {{REPORT_NAME}}.]]
   - Noise check (spec "Noise filter"): items added this week that look like excluded, automated or bulk mail, each with its sender, so {{OWNER_SHORT}} can extend a signature or rule. Omit the section when there are none.
5. {{REPORT_NAME}} draft: follow spec "{{REPORT_NAME}} drafting" exactly. Write it to `status-reports/YYYY-MM-DD.md` at the kit root (today's date; create the folder if missing; never overwrite, append -2 instead). Title: {{REPORT_TITLE_FORMAT}} for the REPORT PERIOD. Audience: {{REPORT_AUDIENCE}}. The draft is the last sent {{REPORT_NAME}} edited for this week, not a fresh write-up. The "For you" sections go only in the report doc, never in the file.
6. Run the spec's Retention step now, so its counts can go in the review.
7. Save to the database in one pinned ArtifactData batch (spec "Reports"): `reports/<today>-review` (sections: one per initiative, Drive first, then "Dormant FYI initiatives", "Open items older than 5 workdays", "Load check", "Noise check" and "Retention" with the step 6 counts; `item_id` on every line about an item)[[IF report:  and `reports/<today>-status` (title, period_start, period_end, file_path, the configured sections followed by the "For you" sections, `body_md` = the exact file text from step 5)]]. Read back and confirm headline, sections and body_md persisted.
8. End with: the review headline,[[IF report:  the {{REPORT_NAME}} file path,]] the {{NAME}} link ({{ARTIFACT_URL}})[[IF report: , and a one-line reminder that {{OWNER_SHORT}} sends the {{REPORT_NAME}} personally; {{NAME}} never emails it]]. If the step 7 batch failed, print the full review in chat and say it was not saved.

If any connector fails, say which and that its data is unavailable (not zero).
