---
name: connect-plant-data
description: Gets a plant's machine signals into InduQT so Claude can see them — browses an OPC UA server, previews an MQTT broker, creates a site-scoped ingest key with a ready Node-RED flow for PLC gateways, historians and proprietary feeds, or loads CSV/Excel history from a link — then sets units and normal ranges and adds the machine's manuals. Registers InduQT as the sensor source for Claude for Manufacturing, so its breakdown, report and CAPA skills read live data instead of historian exports. Use when someone says "connect our machines," "get our PLC data in," "we have an OPC UA server," "hook up our MQTT broker," "load this sensor history," "connect our historian," or "the plugin can't see our sensor data."
---

# Connect plant data to InduQT

Read `../../shared/induqt-basics.md` first.

Claude for Manufacturing reads sensor data from historian exports when it has
nothing better. InduQT is a live source that already has an MCP connector, so
in its `custom-connector` ladder this is the top rung: **a vendor MCP exists,
nothing needs building.** This skill does the connecting.

## Steps

### 1. Where things stand

`get_started`, then `list_sites` and `list_machines`. If the plant and the
machine already exist and data is arriving (`lastDataAt` recent), say so and
skip to step 5.

If the user has no machines yet and only wants to see how it works,
`list_demo_machines` shows live public demos (a boiler on OPC UA, trams over
MQTT, a tide gauge over CSV).

### 2. Site and machines

- `create_site` once per plant, with its timezone. The timezone decides how
  every time is shown to the plant.
- `create_machine` per asset, with family (pumps, compressors, vfd, cnc,
  servo, chillers, boilers, conveyors, panels), make, model, and **the plant's
  own asset tag as `externalId`** — the tag the CMMS and the production report
  use. Without it, records cannot be joined to work orders.

### 3. Pick the route by what the plant has

| The plant has | Route |
|---|---|
| An OPC UA server reachable from the internet | `browse_opcua_server` from the Objects folder down to the machine's tags → `create_connector` type `opcua` with the node ids |
| An MQTT broker reachable from the internet | `preview_mqtt` on the topic filter to see topics, rates and numeric fields → `create_connector` type `mqtt`, mapping the machine to a topic level or payload field |
| Azure Event Hub / IoT Hub | `create_connector` type `eventhub` |
| Node-RED, a PLC gateway, a historian (PI, Ignition, Canary…), anything on the plant network only | `create_ingest_key` for the site → give the user the returned Node-RED flow or curl example; devices POST `{machine, ts, values}` |
| History in Excel or CSV | Publish it at a public https link (e.g. Google Sheets "Publish to web" CSV) → `import_file_url` with the time column and the site's timezone |

For connector `config` and `mapping`, follow https://induqt.ai/docs/connectors.

**Preview long enough.** Plant gateways often publish every 10–30 s. Run
`preview_mqtt` for 15 s; a 5 s preview can see nothing on a healthy topic.

**The ingest key.** Through the connector, `create_ingest_key` returns the
key's prefix and the ingest URL, not the full key. Tell the user to copy the
full key from the InduQT app (Sites → the site → keys) straight into the
gateway; never ask them to paste it into the chat.

**History length.** InduQT compares a window with the machine's own history.
A CSV of a few hours has no baseline: the multivariate checks need about
5–8 hours and the wear trend about 5 days. Ask for at least a few days of
normal running before the period in question.

**Credentials.** Browse and preview take a user name and password for that
call only. Ask for a read-only account. Never repeat a password, key or token
back in the chat, never write one into a file, and remind the user that an
ingest key is shown once and should go straight into the gateway's settings.

**Network.** OPC UA and MQTT here must be reachable from the internet. A
server on the plant LAN only is the ingest-key route, run from inside the
plant.

### 4. Confirm data is flowing

After a few minutes, `latest_values` on each machine. Report which signals
arrived and which did not. A machine with no data after ten minutes is a
mapping problem — re-check the topic, node id or machine field.

### 5. Name signals and set normal ranges

`update_signal` per signal: a readable name, units, `normalLow` and
`normalHigh`. Take ranges from the manual, the nameplate or the plant's
operating limits — ask; never invent a range. The decision engine watches
against these, so a wrong range means wrong alerts.

### 6. Add the manuals and history

`add_documents_from_links` for manuals on the web; `get_upload_link` for PDFs,
work-order exports and photos of nameplates. InduQT reads scanned pages and
cites them in diagnoses.

### 7. Tell Claude for Manufacturing where the data is

When Claude for Manufacturing is installed, record InduQT as the plant's
sensor source in its plant setup (via its `onboarding` or `custom-connector`
skill, whichever owns the connector list): system = InduQT, access = vendor
MCP, scope = the sites and machines connected, asset key = `externalId`.
From then on its `breakdown-diagnosis`, `root-cause-capa` and
`daily-production-report` read the signals through this plugin's other skills.

## Output

A short table: machine, asset tag, route, signals flowing, ranges set,
manuals added — and what is still missing, with who has to provide it.
