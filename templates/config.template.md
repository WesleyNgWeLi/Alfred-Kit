# Assistant configuration

SETUP.md writes this file to `config.md` at the kit root from the setup interview. It is the single source for every generated file. To change anything later, open Claude Code in this folder and say "Read SETUP.md and reconfigure"; do not hand-edit generated files.

Values in `<angle brackets>` are filled during setup. Defaults are shown where one exists.

```yaml
assistant:
  name: Alfred                 # what the page, briefs and tasks call it
  command: alfred              # the item command, typed as /alfred <id>; lowercase, letters/digits/dashes
  task_prefix: alfred          # scheduled task ids become <prefix>-morning, <prefix>-afternoon, <prefix>-weekly
  artifact_url: pending        # setup replaces this with the page URL after publishing

owner:
  full_name: <e.g. Sam Lee>
  short_name: <what briefs call you, e.g. Sam>
  email: <work address; used only to find your own sent mail and past reports>
  role: <e.g. Product manager, Payments>
  manager: <name, or blank>
  timezone: <IANA name, e.g. America/Chicago>
  workdays: Mon-Fri

routines:                      # 24h, in owner.timezone. Setup converts to this machine's clock for the schedule. The app adds a few minutes of jitter.
  machine_timezone: <this computer's timezone, detected at setup>
  morning:
    enabled: true              # always on: the morning sweep is what keeps the data current
    time: "07:30"
    days: Mon-Fri
    joke: true                 # open the brief with one work-safe joke
  afternoon:
    enabled: true
    time: "16:00"
    days: Mon-Fri
    label: 4 PM check-in       # tab and task title
  weekly:
    enabled: true
    day: Fri
    time: "15:00"
    label: Weekly review
  first_sweep_lookback_days: 7 # how far back the very first sweep reads

weekly_report:                 # a draft the weekly run writes for you to send yourself; never emailed
  enabled: true
  name: Status report          # e.g. Weekly update, Friday notes, 5/15
  title_format: "Status | MM/DD - MM/DD"
  title_period: upcoming       # upcoming = MM/DD are the first and last workday of next week; completed = this week (the week the weekly run falls in)
  audience: <e.g. my manager>
  sections:                    # in order; each has a heading, what goes in it, and a role
    - heading: Priorities
      prompt: What I will focus on next week, by initiative
      role: priorities         # priorities | accomplishments | blockers | info | other
      self_written: false      # true = left empty in the draft; suggestions go to "For you: to consider"
    - heading: Accomplishments
      prompt: What I finished this week (verified evidence only)
      role: accomplishments    # exactly one section has this role (verified-evidence rules apply)
      self_written: false
    - heading: Challenges
      prompt: What is blocking me or slowing me down
      role: blockers
      self_written: false
    - heading: Information
      prompt: OOO, things waiting on my manager
      role: info               # dropped Drive items are listed as "Deprioritized" here (info, else blockers, else the last section)
      self_written: false
  previous_report_search:      # outlook_email_search arguments that find your sent ones (no folderName: search all folders, then keep only hits you sent); omit to skip
    query: <e.g. Status>
    recipient: <the address it goes to, e.g. your manager's>

initiatives:                   # "drive" = you own it, track closely and nudge; "fyi" = track, surface only direct asks to you
                               # Tier here only seeds the page. After setup, move initiatives with the page's "Drive this" / "Move to FYI" buttons
                               # and hide finished ones with Archive; reconfigure never overwrites those choices.
  drive:
    - id: <kebab-case-id>
      name: <display name>
      signals: <people, projects, vendors, keywords that mean a message belongs here>
  fyi:
    - id: admin
      name: Admin and HR
      signals: benefits, timesheets, training, IT requests
  allow_new_fyi: true          # a recurring topic with 2+ messages that fits none becomes a new FYI initiative (announced in the brief)

mail:
  excluded_folders: []         # never read; matched by sender/subject, not by folder lookup, e.g.
                               #   - {folder: Newsletters, signature: "an unsubscribe link, or a list sender you never wrote to"}
                               #   - {folder: Alerts, signature: "sender alerts@monitoring.example.com, or [ALERT] in the subject"}
                               #   - {folder: Low priority, signature: none}   # no single pattern; only a direct ask gets through
  summary_folders: []          # summarize changes only, never create items, e.g.
                               #   - {folder: Jira, hours: 24, routine: morning, label: "Jira, last 24h", signature: "sender jira@yourco.atlassian.net, or [JIRA] in the subject"}
                               # signature = how sweeps spot this mail in the all-folder results so it never becomes an item
  vip_senders: []              # never filtered, always read, treated as Drive-level asks; "Name <address>" each, e.g. ["Dana Ortiz <dana.ortiz@example.com>"]
  vip_unread_check_days: 14    # morning sweep also looks back this far for unread VIP mail you never answered
  noise_senders: []            # skip entirely, e.g. [noreply@vendor.com, newsletters@*, "Notification-*"]
  noise_rules:                 # plain-language skip rules
    - Automated notifications (CI, ticketing, calendar responses, OOO replies) unless they ask you a question by name
    - Marketing and bulk mail with an unsubscribe link
    - Social chatter (lunch, thanks, emoji-only)
  ignore_personal: true        # personal/family mail is never logged
  recap_senders: []            # "Name <address>" of people whose meeting-recap emails assign you tasks; every task owned by you becomes a commitment
  check_drafts_and_outbox: true

teams:
  enabled: true

confidentiality: []            # hard filters, e.g. "Never mix details between clients"

style:                         # claims are always labeled VERIFIED / INFERRED / UNKNOWN
  no_em_dashes: true
  extra: <any other preference, e.g. "bullets under 15 words"; blank = none>
```
