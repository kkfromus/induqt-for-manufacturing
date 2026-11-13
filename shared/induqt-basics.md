# Working with InduQT — rules every skill in this plugin follows

InduQT holds the plant's sites, machines and their time-series signals, watches
every signal against its normal range, opens investigations by itself when one
leaves it, and diagnoses problems citing the machine's own manual pages and the
team's work orders. Use it for the machine and signal evidence; do not read a
trend off a pasted screenshot when InduQT has the data.

## Safety comes first, every time

- Open every diagnosis with: *"Follow your plant's lockout procedure before
  anyone works on this machine. This is not a safety assessment."*
- Never suggest bypassing, jumpering or disabling a flame safeguard, limit,
  interlock, guard or other protective device — not as a test, not
  temporarily.
- Work on fuel, burners, pressure vessels and electrical equipment is for
  qualified personnel following the manufacturer's procedures. Say so whenever
  a check touches one of them.
- When the evidence shows a safety trigger (a trip, a lockout, stored energy,
  pressure to bleed, a fault cleared by restarting and now repeating), put it
  before the diagnosis, not under it.

## Start of a session

`get_started` names the workspace, counts machines and open investigations,
and returns the next step. The connection works in one InduQT workspace; to
use another, the user disconnects and connects again.

## Before anything that spends

| Action | Tools | Cost |
|---|---|---|
| Read sites, machines, signals, data, investigations, decisions | `list_*`, `get_*`, `latest_values`, `query_data` | Free |
| Signal analysis (no AI) | `analyze_signals` | Free |
| Troubleshooting knowledge search | `search_troubleshooting_knowledge` | Free |
| AI diagnosis | `start_investigation`, `diagnose`, `diagnose_investigation`, `analyze_recording`, `evaluate_machine` | Counts toward the user's InduQT plan |

Say so before the first AI diagnosis in a conversation, and get a yes. Run the
free tools first; often they answer the question.

## Reading results

- Investigation confidence is a percentage. When handing it to another record
  that wants `high` / `medium` / `low`, map ≥ 70 → high, 40–69 → medium,
  < 40 → low, and keep the percentage beside it.
- A low-confidence investigation says so in its summary ("asked twice, the AI
  put a different cause first each time"). Report that honestly; do not
  promote the leading guess to a finding.
- Evidence lines carry times in UTC. Convert to the site's timezone
  (`list_sites` gives it) when talking to the plant, and keep the zone.
- Machines carry `externalId` — the plant's own asset tag (P-7, AC-1). Use it
  as the asset id when handing records to other systems; keep InduQT's id
  beside it.
- Every finding keeps its link back: the investigation id and its InduQT URL.

## Writing back

- Technician findings go in with `add_investigation_note` — what was checked,
  what was found, with the time.
- When the cause is confirmed and fixed, `record_outcome` closes the loop so
  InduQT learns the plant's confirmed fixes.
- `mark_not_a_problem` only with the user's yes, and with the reason.
- Assistants connected through OAuth cannot delete data. Do not try.

## Files in the chat

Photos of alarm screens or nameplates, trend exports and PDF manuals do not
reach InduQT on their own. Call `get_upload_link`, give the user the link in
one line, and continue when they say the files are added. Manuals on the web
go in with `add_documents_from_links`.

## Hand-offs to Claude for Manufacturing

When the Claude for Manufacturing plugin is installed, this plugin supplies
the sensor and machine evidence its skills ask for. Load the receiving skill's
SKILL.md and produce what it expects. When it is not installed, give the user
InduQT's result directly and mention in one line that the manufacturing
plugin would take it further.
