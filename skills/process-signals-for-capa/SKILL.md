---
name: process-signals-for-capa
description: Lines up a machine's process signals from InduQT against a quality problem — the run that made the bad lot, the hours before an inspection failure, a scrap spike — finds which signals drifted or changed regime and when, compares with good runs and peer machines, and hands a verified timeline to Claude for Manufacturing's root-cause-capa, ncr-writeup and inspection-to-capa skills. Use when someone says "was it the machine," "what was the process doing when we made these parts," "pull the sensor data for this lot," "did temperature or pressure drift," or "root cause this defect."
---

# Process signals for root cause and CAPA

Read `../../shared/induqt-basics.md` first.

A nonconformance names a lot, a time window and a machine. Whether the
process drifted in that window is a question the signals can answer, and the
answer either supports a process cause or rules it out.

## Steps

1. **Pin the window** — the machine (by tag, `externalId`), and the start and
   end of the run that made the suspect parts, in the site's timezone. Take
   them from the NCR, the production work order or the lot record. If the
   window is not known, ask; do not guess it from the defect.

2. **A good window to compare** — a run of the same part on the same machine
   that passed, ideally recent. Ask for it, or take the last comparable run
   the user names.

3. **Read both windows** (free):
   - `query_data` for the suspect and the good window, same signals.
   - `analyze_signals` on the suspect window: drift (CUSUM, EWMA), change
     points, regime shifts, oscillation, peer comparison and matches to past
     events.

4. **Build the timeline** — each finding with its time, signal, direction and
   size, side by side with the good run. Mark which findings fall **before**
   the first bad part (candidate causes) and which fall after (likely
   effects). Note signals that did not move: ruling out a process cause is
   also a finding.

5. **Check what InduQT already knew** — `list_investigations` for the machine
   in and just before the window. A detection InduQT raised during the run
   that nobody acted on is part of the story, stated plainly.

6. **Hand off** — with Claude for Manufacturing installed, give
   `root-cause-capa` (or `inspection-to-capa` when the full chain is running)
   the timeline as its sensor evidence: machine tag, window, each finding with
   time and source (InduQT machine id and the analysis), and the good-run
   comparison. It combines this with inspection data, maintenance events and
   lot and shift clustering, and writes the 8D. When a finding points at
   purchased material rather than the process, say so — that is its
   supplier-caused flag to raise, not this skill's.

## Never

- Call a correlation a cause. The timeline is evidence for and against;
  `root-cause-capa` ranks the causal chains.
- Change signal ranges to make a window look normal.
