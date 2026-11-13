---
name: machine-down-diagnosis
description: When a machine trips, alarms or goes down, pulls its live signals from InduQT — which signal moved first, when, and in which direction — runs InduQT's signal analysis and, with the user's yes, its AI diagnosis citing the machine's manuals and past work orders, then hands the evidence to Claude for Manufacturing's breakdown-diagnosis and emits the downtime record for the next production report. Takes the technician's findings back into InduQT and closes the loop with the confirmed cause. Use when someone says "the machine is down," "we got a fault code," "the pump tripped," "why did the compressor stop," "what does this alarm mean," or "diagnose this machine."
---

# Machine down → diagnosis → downtime record

Read `../../shared/induqt-basics.md` first. **Its safety rules lead every
answer this skill gives.**

## Steps

### 1. Capture the event

Machine (find it with `list_machines`; match the plant's tag to
`externalId`), time it went down, the alarm or fault code verbatim, and what
the machine was doing. If the machine is not in InduQT, still help: go to
step 3 with the equipment family.

### 2. Is InduQT already on it?

`list_investigations` and look for the machine near that time. InduQT's
monitoring often opened one before anyone called. Read it with
`get_investigation`. Several open investigations on one machine with the same
cause are one problem, not several.

### 3. Fault code meaning (free)

`search_troubleshooting_knowledge` with the code, make and model. It returns
the meaning, causes with what points to each, and the check that confirms
each, with sources. Search with the fault code and model first; a long
symptom sentence often finds nothing. When it finds nothing, say so, and
label anything from general practice as such — never present it as
InduQT's.

### 4. Read the signals (free)

- `latest_values` for the state now, with out-of-range flags.
- `query_data` for the window before the event (default: the two hours
  before; widen when the symptom built up slowly).
- `analyze_signals` over the same window: change points, anomalies, peer
  comparison, matches to past events, and the time until a signal reaches its
  limit. No AI diagnosis is used.

State **which signal moved first, at what time (site timezone), and which way**.
That ordering is usually the strongest single piece of evidence.

Read `analyze_signals` by its `summary`, `results` and `failure` blocks; the
forecast point series is long and rarely needed. Two cautions:

- **A repeating event is not wear.** When the forecast's crossing comes from
  spikes at a fixed interval (every 6 h, every shift) and the degradation
  trend is flat, report a recurring event on a schedule — ask what runs at
  that time (a tank change, CIP, a product changeover) — rather than the
  wear cause the forecast names.
- **Check the readings against the ranges yourself.** A machine's
  `latestSeverity` can read `info` while `query_data` shows a signal outside
  its `normalLow`–`normalHigh`. The readings win.

### 5. AI diagnosis (ask first)

If the free evidence does not settle it, say that a diagnosis counts toward
the InduQT plan and ask. On a yes: `start_investigation` with the symptom in
the user's words and `at` set to the event time — or
`diagnose_investigation` on the one InduQT already opened.

The diagnosis reads only the **two hours before `at`**. For a fault that
built up slowly, those two hours are already inside the fault, so the
baseline is wrong and rises look small (a vibration that tripled reads as
"+20 %"). Use the onset `analyze_signals` found: set `at` to about two
hours after the onset, and report the size of the change from
`analyze_signals`, not from the diagnosis card.

Report: likely cause with its percentage, the evidence with times, other
causes, the first actions, and the **single next check that best separates
the leading causes**. When InduQT says it is not sure ("asked twice, a
different cause first each time"), say exactly that.

### 6. Hand off to Claude for Manufacturing

With Claude for Manufacturing installed, load its `breakdown-diagnosis`
SKILL.md and supply the sensor window it asks for from InduQT: the signals,
the first-mover, the analysis findings and the investigation link. It adds
the CMMS history and photos, ranks causes and drafts the PM change. Do not
duplicate its ranking; InduQT's diagnosis is one input to it.

Then emit the downtime record (its contract `downtime-record`, v3) for
`daily-production-report`:

| Field | From |
|---|---|
| `asset_id` / `asset_id_verbatim` | `externalId` / the name the plant used |
| `started_at` | the event time, with the site's zone |
| `ended_at`, `duration_minutes` | null until the machine is back; never zero |
| `reason_code` | the plant's own code, not InduQT's cause text |
| `source_system` | `breakdown_diagnosis` |
| `source_ref` | the InduQT investigation URL |
| `leading_cause` | the leading cause |
| `leading_cause_confidence` | percentage mapped per the basics file, percentage kept in the text |
| `safety_flagged` | true when any safety trigger in the basics file shows; say which |

### 7. Close the loop

- Technician findings → `add_investigation_note`, then
  `diagnose_investigation` to update.
- When the fix is confirmed: `list_decisions` for the machine and pick the
  decision whose `from`–`to` window covers the event and whose `kind` is not
  `normal` (decisions do not carry the investigation id; match on time). Its
  `final.causes` holds the hypothesis ids (for example
  `cavitation_or_starvation`). Then `record_outcome` with that decision id,
  the confirmed hypothesis id (or `no_fault` / `other`), the check that found
  it and a note. This is what makes InduQT's next diagnosis on this machine
  better. The decision list is large; read it once and keep only the
  matching entry.

## Never

- Suggest bypassing, jumpering or disabling any protective device.
- Send anyone to the machine before the lockout line and any safety flag.
- Present a low-confidence cause as the answer.
