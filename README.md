# EM Weekly Report Skill

Automated weekly leadership reports for Razorpay Engineering Managers. Scans your team's Slack activity, extracts highlights/callouts/links, and saves a structured markdown report — all driven by a single config file.

## What It Does

Every week, run `/weekly-report` in Claude Code and get:

- **Highlights** — key achievements grouped by workstream, with precise numbers and PR links
- **Callouts / Help Needed** — blockers, risks, and items needing leadership attention
- **Media & Links** — Loom recordings, Figma links, PR links collected from Slack

Output is a clean markdown file in the repo, ready to copy into your leadership doc or share directly.

## Prerequisites

1. **Claude Code** — [Install Claude Code](https://docs.anthropic.com/en/docs/claude-code) if you haven't already
2. **Slack MCP** — An MCP server that provides Slack search access (see [MCP Setup](#mcp-setup))

## Quick Start

```bash
# 1. Clone this repo
git clone git@github.com:razorpay/em-weekly-report-skill.git my-team-reports
cd my-team-reports

# 2. Configure MCP servers
cp .mcp.json.example .mcp.json
# Edit .mcp.json with your Slack token (see MCP Setup below)

# 3. Run the interactive setup
claude /setup

# 4. Generate your first report
claude /weekly-report
```

## MCP Setup

### Slack MCP

The skill needs a Slack MCP server to search messages. Copy `.mcp.json.example` to `.mcp.json` and configure:

```json
{
  "mcpServers": {
    "slack": {
      "command": "npx",
      "args": ["-y", "@anthropic/slack-mcp"],
      "env": {
        "SLACK_TOKEN": "<your-slack-bot-token>",
        "SLACK_TEAM_ID": "<your-slack-workspace-id>"
      }
    }
  }
}
```

**For Razorpay engineers:** If you already have the Razorpay Slack MCP configured in your Claude Code settings (from rzp-discover or similar), you can skip this step — the skill will use whatever Slack MCP is available.

## Configuration

All team-specific values live in `team-config.yaml`. Run `/setup` for a guided walkthrough, or copy the example and edit manually:

```bash
cp team-config.yaml.example team-config.yaml
```

### Config Reference

| Section | Field | Required | Description |
|---------|-------|----------|-------------|
| `team.name` | string | Yes | Team name (e.g., "Checkout") |
| `team.em_name` | string | Yes | Your name |
| `slack.members` | list | Yes | Team members with `name` and `slack_id` |
| `slack.channels` | list | Yes | Channels to scan with `name` and `id` |
| `slack.workstreams` | list | No | Active focus areas for grouping highlights |
| `report.sections` | list | No | Sections to include: highlights, callouts, media_links |
| `report.tone` | string | No | Writing style guidance |
| `report.output_dir` | string | No | Local output path template |

### Finding Slack IDs

- **Member ID**: Click a person's Slack profile → "..." menu → "Copy member ID"
- **Channel ID**: Right-click a channel → "View channel details" → scroll to the bottom

## Usage

### Generate a weekly report

```bash
claude /weekly-report
```

This scans the last 7 days of Slack activity by default.

### Specify a date range

```bash
claude /weekly-report for March 17-23
```

### Update your config

Edit `team-config.yaml` directly, or re-run the setup wizard:

```bash
claude /setup
```

## Output

Reports are saved as markdown at:

```
output/{start_date}_to_{end_date}/reports/weekly_report_{end_date}.md
```

## Example

See `examples/frontend-platform.yaml` for a complete config from the Frontend Platform team.

## Project Structure

```
.
├── CLAUDE.md                          # Project instructions for Claude Code
├── README.md                          # This file
├── team-config.yaml.example           # Annotated config template
├── .mcp.json.example                  # MCP server config template
├── .claude/
│   └── commands/
│       ├── weekly-report.md           # /weekly-report command
│       └── setup.md                   # /setup interactive wizard
├── examples/
│   └── frontend-platform.yaml        # Reference config
└── output/                            # Generated reports (gitignored)
```

## FAQ

**Q: Can I use this with Cursor instead of Claude Code?**
A: The commands are written for Claude Code's `.claude/commands/` format. For Cursor, you'd need to adapt them into `.cursor/skills/` format. The logic is identical — only the file structure differs.

**Q: What if I don't have a Slack MCP?**
A: The `/setup` command will detect this and guide you through configuration. You need at least read access to search messages in your team's channels.

**Q: Can I add custom metrics (PostHog, Cursor Analytics, etc.)?**
A: This base skill focuses on Slack scanning + report generation. For custom metrics, extend the `/weekly-report` command with additional steps, or create a separate command that merges metrics with the Slack report.

**Q: How do I update workstreams when priorities change?**
A: Edit the `slack.workstreams` list in `team-config.yaml`. No restart needed — the skill reads the config fresh on every run.
