# AGENTS.md — AI Agent Repository Instructions

How AI agents (Claude Cowork, Claude Code, any LLM tool) should work with this repo correctly.

---

## What This Repo Is

The **Claude Token Command Center** — a single self-contained HTML file
(`claude-token-dashboard.html`) that a Cowork scheduled task (`Scheduled/SKILL.md`) refreshes daily.
It shows live plan limits, exact per-day token burn (Claude sources only), the heaviest sessions, and
which skills / MCP connectors are in use.

> **This is one of two dashboards — never collide them.**
> - This repo owns `claude-token-dashboard.html` (Command Center). It is **Claude-sources-only**
>   (Cowork + Claude Code + Chat-est) — **no Codex**.
> - The separate [Token Burn Dashboard](https://github.com/edmatibag9Dev/Token-Burn-Dashboard-)
>   (`token-burn-dashboard.html`) is the heatmap of burn across **all** sources, **including Codex**.
> - For ~30 days the two overwrote each other because both wrote `claude-token-dashboard.html`. The
>   fix is one writer per file plus an identity guard (below).

## Ownership & identity guard

- The daily task writes **only** `~/Documents/Claude/claude-token-dashboard.html` and must verify it
  contains `<h1>Claude Token Command Center</h1>` before editing. If the marker is missing, STOP and
  alert — do not write.
- Never write `token-burn-dashboard.html` (owned by the Token Burn project's native launchd pipeline).
- Exact numbers come from the Token Burn project's `daily-burn.json` / `sessions.json` /
  `usage-breakdown.json` (read-only here); only the native ingesters write those.

## Primary file: claude-token-dashboard.html (gitignored — embeds real session titles)

The real file is **gitignored**; only `claude-token-dashboard.sample.html` (sanitized placeholder data)
is committed. The dashboard reads these embedded constants, updated on each scheduled run:

- `const USAGE_SUMMARY` — LIVE plan limits scraped from `claude.ai/settings/usage` (not token counts).
- `const DAILY_BURN` — EXACT per-day tokens from `daily-burn.json`. **Claude sources only** — the
  `SOURCES` list excludes Codex and totals are computed Claude-only. Do **not** add a Codex column.
- `const SESSIONS` — EXACT per-session rows from `sessions.json`.
- `const USAGE_BREAKDOWN` — skills + MCP-connector counts from `usage-breakdown.json` (drives the
  Skills usage and MCP connector cards, rendered client-side).
- `const LAST_UPDATED` — render timestamp (America/Los_Angeles).

KPIs, charts, the heat of skills/connectors, and the daily-detail table are all recomputed by the
dashboard's own JS — agents only swap the constants above.

## Scheduled task: Scheduled/SKILL.md

The complete daily-refresh instructions (runs ~5:02 PM PT). Read it before executing. Key rules:
always use targeted `Edit` replacements (never rewrite the whole file); reconcile each daily-burn row
(`total == cowork + claude_code + claude_chat_est + codex`, cache excluded) before publishing; confirm
the identity marker survives the edits.

## What agents should NOT do

- Do not write `token-burn-dashboard.html`, and do not add Codex to this dashboard.
- Do not commit the real `claude-token-dashboard.html` (gitignored) — only the `*.sample.html`.
- Do not rewrite the full HTML file — always targeted replacements.
- Do not commit tokens, API keys, or credentials.

## Alert protocol

If blocked (login required, identity marker missing, reconciliation fails, MCP unavailable), email
`edmatibag9@gmail.com` via Gmail/Chrome MCP — Subject `⚠️ Claude Token Command Center — [Category]: [issue]`.
Fallback: an Apple Note titled `⚠️ TASK ALERT: Claude Token Command Center — [Date]`. Stop after sending.

---

## Repo Structure

```
claude-token-dashboard.html         ← primary artifact (gitignored — real usage)
claude-token-dashboard.sample.html  ← committed sanitized reference
Scheduled/SKILL.md                  ← daily refresh task (ownership-guarded)
AGENTS.md                           ← this file
CONTRIBUTING.md                     ← commit + README standards
README.md                           ← human-readable overview
.gitignore                          ← keeps real-usage data out of git
```

## GitHub push (for agents making changes)

GitHub Contents API. Owner `edmatibag9Dev`, repo `Claude-Token-and-Command-Center-Dashboard`, branch `main`.
New file: `PUT .../contents/{path}` with base64 content. Update: `GET` the file for its `sha`, then `PUT`
with `sha` (omitting it returns 409). Commit format: see `CONTRIBUTING.md`.
