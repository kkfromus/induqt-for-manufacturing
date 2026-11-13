# InduQT for Manufacturing

Live machine signals and troubleshooting for plants, built to work alongside
Anthropic's **Claude for Manufacturing** plugin.

Claude for Manufacturing diagnoses breakdowns, writes CAPAs and reports the
morning numbers — but it needs to see what the machines were doing. This
plugin connects Claude to InduQT, which holds the plant's live signals from
OPC UA, MQTT, Node-RED, historians and CSV history, watches every signal
against its normal range, and diagnoses problems citing the machine's own
manuals and the team's past work orders.

## Install

In Claude Code:

```
/plugin marketplace add kkfromus/induqt-for-manufacturing
/plugin install induqt-for-manufacturing@induqt
```

On Claude Code 2.1.275 or later this works in one step:

```
/plugin install induqt-for-manufacturing --marketplace kkfromus/induqt-for-manufacturing
```

## Skills

| Skill | What it does |
|---|---|
| `connect-plant-data` | Browses OPC UA, previews MQTT, creates a site-scoped ingest key with a Node-RED flow for gateways and historians, or loads CSV history; sets units and normal ranges; adds manuals; registers InduQT as the plant's sensor source for Claude for Manufacturing. |
| `machine-down-diagnosis` | From an alarm or trip: fault-code meaning, which signal moved first and when, signal analysis and (with a yes) an AI diagnosis; hands the evidence to `breakdown-diagnosis`, emits the downtime record for `daily-production-report`, and records the confirmed fix back in InduQT. |
| `ai-watch-triage` | Groups and ranks the problems InduQT's monitoring found on its own, separates faults from process changes, and feeds the active ones to `morning-plant-pulse` as exceptions. Closes false alarms with a reason so InduQT learns. |
| `process-signals-for-capa` | Lines up a machine's signals against a bad lot or a scrap spike, compares with a good run, and hands a timeline to `root-cause-capa`. |

## Connector

The plugin uses the InduQT MCP server at `https://api.induqt.ai/mcp`. You sign
in to your InduQT account the first time Claude uses it. No API keys are
stored in the plugin. New users can try it on InduQT's public live demo
machines without connecting anything.

## Safety

Every diagnosis opens with a lockout reminder and is not a safety
assessment. The skills never suggest bypassing, jumpering or disabling a
safeguard, limit or interlock, and say when a check is for qualified
personnel only.

## Costs and consent

Reading machines, signals, investigations and the troubleshooting knowledge
base, and the signal analysis, are free. AI diagnoses count toward your
InduQT plan; Claude asks before the first one.

## Works best with

- **Claude for Manufacturing** (Anthropic): breakdown diagnosis, production
  reporting and CAPA. Without it, the skills still deliver InduQT's readings,
  analyses and diagnoses directly.

## Data handling

Machine data, manuals and notes you add are stored in your InduQT workspace and
processed by InduQT. The plugin sends data only to the InduQT connector
declared above. Privacy policy: https://induqt.ai/privacy

## Support

support@induqt.ai · https://induqt.ai/claude
