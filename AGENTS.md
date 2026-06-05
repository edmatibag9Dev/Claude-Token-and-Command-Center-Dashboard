# AGENTS.md — AI Agent Repository Instructions

This file tells AI agents (Claude Cowork, Claude Code, or any LLM-based tool) how to work with this repository correctly.

---

## What This Repo Is

A locally-hosted Claude Command Center dashboard. It is a **single HTML file** (`claude-token-dashboard.html`) that an AI agent updates daily via a Cowork scheduled task defined in `Scheduled/SKILL.md`.

---

## Primary File: claude-token-dashboard.html

This is the only file that the scheduled task modifies. It contains three logical sections:

### Section 1 — Token Usage (dynamic, update every run)
These blocks must be replaced with fresh data on every scheduled run:
- `const REAL_DATA` → replace only the `sessions: [...]` array inside it
- `const USAGE_SUMMARY` → replace the whole object with live values
- `<div class="header-meta">` → update the four meta-item divs
- Summary cards (Current Session %, Weekly Usage %, Extra Usage Spending)
- `<div class="last-updated">` → update with today's date and live data note

### Section 2 — Command Center Config (static, do NOT modify)
These objects are permanent config. Do not overwrite, reformat, or regenerate them during a scheduled run:
- `const KNOWN_SKILLS` — skill detection config
- `const KNOWN_CONNECTORS` — connector detection config
- `function detectSkillUsage()` — skill matching logic
- `function detectConnectorUsage()` — connector matching logic
- `function renderCommandCenter()` — rendering logic

The Command Center auto-derives its output from the `REAL_DATA` sessions array you update in Section 1. You do not need to touch Section 2 to refresh the dashboard.

### Section 3 — Charts and Table (auto-generated)
These are rendered by JavaScript at page load from `REAL_DATA`. Do not modify the chart canvas elements or the sessions table structure.

---

## Scheduled Task: Scheduled/SKILL.md

This file contains the complete step-by-step instructions for the daily 4PM refresh. Read it before executing a run. Key rules:
- Always use targeted `Edit` tool replacements — never overwrite the full HTML file
- If `claude.ai/settings/usage` is inaccessible, use `session_info` estimates only and note it in the header
- Maximum 25 sessions in `REAL_DATA`
- Cost formula for claude-sonnet-4-6: `(input × $0.000003) + (output × $0.000015) + (cache × $0.0000003)`

---

## Session Classification

When building `REAL_DATA`, classify sessions from `session_info` titles as follows:

| Class | Trigger | Tokens (input / output) |
|---|---|---|
| dispatch | title contains "dispatch", "brain", "openbrain" | ~2,000 / 600 |
| simple | short titles, admin tasks | ~5,000 / 1,500 |
| medium | reports, analysis, journal entries | ~25,000 / 5,000 |
| heavy | scraping, automation builds, multi-source tasks | ~80,000 / 8,000 |

---

## Alert Protocol

If the task is blocked for any reason, send an email to `edmatibag9@gmail.com` using Gmail via Chrome MCP. Subject format:

```
⚠️ Claude Command Center — [Category]: [One-line issue]
```

Fallback: write an Apple Notes entry titled `⚠️ TASK ALERT: Claude Command Center — [Date]`.

**Stop and do not retry after sending the alert.**

---

## What Agents Should NOT Do

- Do not regenerate or reformat `KNOWN_SKILLS` or `KNOWN_CONNECTORS`
- Do not rewrite the full HTML file — always use targeted replacements
- Do not commit tokens, API keys, or credentials to this repo
- Do not change the dashboard's CSS or chart library version during a data refresh run
- Do not modify `renderCommandCenter()` unless explicitly instructed to change detection logic

---

## Repo Structure

```
claude-token-dashboard.html    ← primary artifact, updated daily
Scheduled/SKILL.md             ← scheduled task instructions
AGENTS.md                      ← this file
CONTRIBUTING.md                ← commit and README standards
README.md                      ← human-readable project overview
```

---

## GitHub Push Instructions (for agents making code changes)

Use the GitHub Contents API. Owner: `edmatibag9Dev`. Repo: `Claude-Token-and-Command-Center-Dashboard`. Branch: `main`.

- **New file:** `PUT /repos/{owner}/{repo}/contents/{path}` with base64 content
- **Update existing file:** First `GET` the file to retrieve its `sha`, then `PUT` with `sha` included — omitting `sha` on an update returns a 409 conflict error

Commit message format: see `CONTRIBUTING.md`.
