# Claude Command Center Dashboard

A locally-hosted, self-updating intelligence dashboard for Claude Cowork usage. It tracks token consumption, identifies which AI skills are active or idle, and shows which MCP connectors are being used — all from a single HTML file that refreshes automatically every day at 4PM via a Cowork scheduled task.

---

## Overview

This project solves a practical problem: when you use Claude Cowork heavily across many skills and automations, it becomes hard to know what's actually being used, what's costing money, and what's sitting idle. The Command Center gives you that visibility in one place, locally on your machine, no server required.

---

## Features

- **Token Usage Summary** — Live current-session %, weekly limit %, and extra usage spend pulled from `claude.ai/settings/usage`
- **Session Breakdown** — Last 25 Cowork sessions classified by type (cowork / dispatch / chat) with estimated input, output, and cache tokens plus cost per session
- **Skills Command Center** — Every installed skill shown as a card: session count, estimated token cost, last-used date, and active / recent / inactive status. Includes a horizontal bar chart sorted by usage frequency
- **MCP Connector Usage** — Detects which connectors (Open Brain, Day One, Chrome, Gmail, Slack, Apple Notes, Session Info) appear in session activity
- **Daily Auto-Refresh** — A Cowork scheduled task runs every day at 4PM, scrapes live data, and writes targeted edits to the HTML file. Static config is never overwritten
- **Email Alert Protocol** — If the scheduled task is blocked (login required, MCP error, Chrome unavailable), it emails `edmatibag9@gmail.com` with a structured alert and falls back to Apple Notes

---

## How It Works

```
claude.ai/settings/usage  ──►  Chrome MCP scrape  ──►  USAGE_SUMMARY block
session_info (last 50)    ──►  classify + estimate  ──►  REAL_DATA sessions array
                                                          │
                                                          ▼
                                               renderCommandCenter()
                                                 ├── detectSkillUsage()
                                                 └── detectConnectorUsage()
```

The dashboard file is a single self-contained HTML file. All skill and connector detection logic is client-side JavaScript that runs in the browser from the sessions array — no backend, no database.

---

## File Structure

```
├── claude-token-dashboard.html        # The Command Center (open this in a browser)
├── Scheduled/
│   └── SKILL.md                       # Scheduled task instructions for the daily 4PM refresh
├── AGENTS.md                          # AI agent instructions for this repo
├── CONTRIBUTING.md                    # Commit standards and README requirements
└── README.md                          # This file
```

---

## Setup

1. Clone or download this repo to your local machine
2. Open `claude-token-dashboard.html` in any browser — it works fully offline
3. To enable daily auto-refresh, the `Scheduled/SKILL.md` task must be registered in Claude Cowork's scheduled task system pointing to this file path
4. Ensure Chrome is open and you are logged into `claude.ai` before the 4PM scheduled run

---

## Configuration

Two JavaScript config objects in `claude-token-dashboard.html` control the Command Center:

**`KNOWN_SKILLS`** — list of skills to track, each with:
- `label` — display name
- `keywords` — strings to match against session titles
- `category` — grouping label (Automation, Sales, Trading, etc.)
- `icon` — emoji displayed on the card

**`KNOWN_CONNECTORS`** — list of MCP connectors to detect, each with:
- `label` — display name
- `keywords` — strings to match against session titles
- `color` — hex color for the count display

To add a new skill or connector to tracking, add an entry to the relevant array. Do not modify `renderCommandCenter()`, `detectSkillUsage()`, or `detectConnectorUsage()` unless you are changing detection logic for all skills.

---

## Scheduled Automation

The daily refresh task (`Scheduled/SKILL.md`) follows this 7-step process:

1. Navigate to `claude.ai/settings/usage` via Chrome MCP
2. Extract billing period, token totals, and usage percentages
3. Pull last 50 Cowork sessions via `session_info` and classify each as simple / medium / heavy
4. Build a 25-session array with estimated tokens and costs
5. Construct a `USAGE_SUMMARY` block with aggregate stats
6. Write targeted edits to `claude-token-dashboard.html` — sessions array, usage summary, header meta, summary cards, and last-updated timestamp
7. Output a completion summary including active/inactive skills and top connector

If any step fails, the task emails `edmatibag9@gmail.com` with a structured alert (category, issue, steps completed, action needed) and stops without retrying.

---

## Data & Privacy

All data stays on your local machine. The HTML file reads no remote APIs at runtime — all data is embedded as JavaScript variables that the scheduled task writes. The only outbound connection is the daily scrape of `claude.ai/settings/usage`, which requires an active browser session.

---

## Contributing

See `CONTRIBUTING.md` for commit message format, README update requirements, and GitHub push instructions using the Contents API.

---

## License

Private repository — for personal use by Ed Matibag.
