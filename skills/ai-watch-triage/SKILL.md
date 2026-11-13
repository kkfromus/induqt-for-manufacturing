---
name: ai-watch-triage
description: Triages the problems InduQT's monitoring found on its own across a plant — groups repeat detections into one problem per machine and cause, ranks them by severity and how close a signal is to its limit, separates real faults from process or demand changes, and drafts the exceptions for Claude for Manufacturing's morning plant pulse and production report. Closes false alarms with the user's yes so InduQT learns. Use when someone says "what did InduQT find overnight," "what's InduQT flagging," "which machines need attention," "triage the alerts," "too many alerts," or as part of the morning report.
---

# Triage InduQT's live detections

Read `../../shared/induqt-basics.md` first.

InduQT's decision engine opens an investigation whenever a signal leaves its
normal range. On a busy plant that is dozens a day, and the same machine
often gets the same cause again and again. A raw list is noise; this skill
turns it into a short list a supervisor can act on.

## Steps

1. **Scope** — the site (`list_sites`) and the window: since the last shift
   change, overnight, or a date the user gives. Use the site's timezone.

2. **Collect** — `list_investigations`; keep those for the site, created or
   updated in the window. `list_machines` for the site gives each machine's
   tag (`externalId`), its last summary and open count.

3. **Group** — one problem = one machine + one likely cause. Count how many
   detections it took and the first and latest time. A machine with three
   "feedwater supply" detections is one problem seen three times, which
   matters more than three separate problems.

4. **Rank**:
   1. Severity `critical`, then `act`, then `watch`, then `info`.
   2. Within a level, a forecast that a signal reaches its limit soonest
      (the summary says "reaches its high limit … in about 4.3 h").
   3. Then repeat count.
   Causes that read "process or demand change, not an equipment fault" go to
   a separate short list: they are about how the line ran, not a repair.

5. **Check the top few** (free) — `get_investigation` for evidence and the
   next best check; `latest_values` to see whether it is still happening now.
   Mark each problem **still active** or **cleared since**. Judge that from
   the readings against their normal ranges, not from the machine's
   `latestSeverity`, which can lag.
   A problem that comes back at the same times every day (every 6 h, each
   shift) is usually a scheduled event on the line; list its times and ask
   what runs then.

6. **Report** — a short table per problem: machine and tag, cause and
   confidence, severity, times seen, first and latest time, still active or
   cleared, the next best check, and the InduQT link. Lead with anything
   critical or safety-related, using the safety wording in the basics file.

7. **Hand off** — with Claude for Manufacturing installed, give the problems
   still active to its `morning-plant-pulse` or `daily-production-report` as
   exceptions needing a decision today, each with its machine tag and link.
   A machine that actually stopped goes through `machine-down-diagnosis` to
   produce a downtime record; a detection alone is not downtime and is never
   logged as one.

8. **False alarms** — when the user says a detection was not a problem, ask
   for the reason and, on their yes, `mark_not_a_problem` with it. When too
   many alerts come from a range that is set wrong, suggest fixing the range
   with `update_signal` instead (from the manual or the plant's limits).

## Never

- Count a detection as downtime.
- Close an investigation as not a problem without the user's yes and reason.
- Run an AI diagnosis on every detection — ask first, and only for the ones
  that matter.
