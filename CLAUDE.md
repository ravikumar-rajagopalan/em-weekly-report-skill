# EM Weekly Report Skill

This project generates weekly leadership reports for Engineering Managers at Razorpay by scanning team Slack activity and saving structured updates as markdown files.

## How It Works

1. You configure your team details once in `team-config.yaml` (roster, channels, workstreams)
2. Run `/weekly-report` each week — Claude scans Slack, extracts highlights/callouts/links, and saves a formatted report
3. Run `/setup` for an interactive walkthrough to create your config

## Project Structure

- `team-config.yaml` — your team-specific configuration (created via `/setup` or from the example)
- `.claude/commands/weekly-report.md` — the weekly report generator command
- `.claude/commands/setup.md` — interactive config wizard
- `output/` — generated markdown reports organized by date range

## MCP Requirements

This skill requires one MCP server configured in `.mcp.json`:

- **Slack MCP** — for searching team messages. Must have read access to the channels listed in your config.

If the Slack MCP is not configured, the skill will tell you what to set up.

## Config File

All team-specific values live in `team-config.yaml`. The skill reads this file at the start of every run. See `team-config.yaml.example` for the full schema with annotations.

Key sections:
- `team` — name and EM identity
- `slack.members` — people whose messages to scan
- `slack.channels` — channels to scan
- `slack.workstreams` — themes for grouping highlights
- `report` — output preferences

## Writing Style

Reports use first person plural ("We", "Our team"). Lead with outcomes, back with data. Numbers from Slack are preserved exactly — never rounded or approximated.
