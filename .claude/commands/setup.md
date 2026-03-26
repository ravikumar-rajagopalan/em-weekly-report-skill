# Setup — Configure Your Team's Weekly Report

Interactive wizard to create `team-config.yaml` for the weekly report generator.

## Pre-flight Checks

Before starting, verify:

1. **Slack MCP is connected.** Run a test search to confirm:
   - Use the Slack MCP `search` tool with query `"test"` and `limit: 1`
   - If it fails, tell the user to configure `.mcp.json` (copy from `.mcp.json.example`) and restart Claude Code

2. **Check for existing config.** If `team-config.yaml` already exists, show its contents and ask:
   - "You already have a config. Do you want to update it or start fresh?"

## Step 1: Team Identity

Ask the user:

```
What is your team name? (e.g., "Checkout", "Onboarding", "Business Banking")
```

```
What is your name? (e.g., "Rizwanul Haque")
```

## Step 2: Team Members

Ask the user:

```
List your direct reports and key contributors (the people whose Slack activity should be scanned).

You can provide names and I'll look up their Slack IDs, or provide both.

Example:
- Pranav Gupta
- Jatin Rathee
- S M Sohaib Alam
```

For each name provided **without** a Slack ID:
1. Search Slack using the MCP `users_list` or `search` tool to find their Slack user ID
2. If multiple matches, show them and ask the user to pick
3. If no match, ask the user to provide the Slack ID manually (instructions: "Click their profile → ⋮ menu → Copy member ID")

Confirm the full roster with the user before proceeding:

```
Team Roster:
1. Pranav Gupta (U0XXXXXX)
2. Jatin Rathee (U0YYYYYY)
3. S M Sohaib Alam (U0ZZZZZZ)

Correct? (or tell me what to change)
```

## Step 3: Slack Channels

Ask the user:

```
Which Slack channels should I scan for team-relevant activity?

These are channels where your team posts updates, standups, or discussions.

Example:
- #checkout-core
- #checkout-standup
```

For each channel name:
1. Look up the channel ID using Slack MCP
2. If not found, ask the user to provide the ID (instructions: "Right-click the channel → View channel details → scroll to the bottom for the Channel ID")

## Step 4: Active Workstreams

Ask the user:

```
What are your team's current active workstreams or focus areas?

These are used to group highlights in the weekly report. You can update them anytime by editing team-config.yaml.

Example:
- Standard Checkout redesign
- Payment Links 2.0
- 1CC consumer app
```

## Step 5: Write Config

Assemble `team-config.yaml` from all the collected information:

```yaml
team:
  name: "{team_name}"
  em_name: "{em_name}"

slack:
  members:
    - name: "{member_1_name}"
      slack_id: "{member_1_id}"
    # ... all members

  channels:
    - name: "#{channel_1_name}"
      id: "{channel_1_id}"
    # ... all channels

  workstreams:
    - "{workstream_1}"
    # ... all workstreams

report:
  sections:
    - highlights
    - callouts
    - media_links
  tone: "confident but not boastful"
  output_dir: "output/{start_date}_to_{end_date}"
```

Write the file to `team-config.yaml` in the project root.

## Step 6: Validate Setup

Run a quick validation:

1. **Slack connectivity**: Search one team member's recent messages to confirm MCP works
2. **Report any issues** and how to fix them

## Step 7: Done

Tell the user:

```
Setup complete! Your team-config.yaml has been created with:
- {N} team members
- {N} channels
- {N} workstreams

To generate your first weekly report:
  /weekly-report

To update your config later, edit team-config.yaml directly or run /setup again.
```
