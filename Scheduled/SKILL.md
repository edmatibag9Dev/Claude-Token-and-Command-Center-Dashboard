---
name: claude-token-dashboard-update
description: Daily 5:02 PM refresh of the Claude Token Command Center (claude-token-dashboard.html ONLY) — live plan limits + exact Claude-source token data + sessions/skills/connectors. Never touches the separate Token Burn dashboard.
---

Refresh the **Claude Token Command Center** dashboard. This task OWNS exactly one file and must never write any other dashboard.

## File ownership contract (critical — this is why the dashboards kept overwriting each other)
- The ONLY file you may write is: `/Users/edmatibag/Documents/Claude/claude-token-dashboard.html` (the Command Center).
- You must NEVER write `/Users/edmatibag/Documents/Claude/token-burn-dashboard.html` — that is the separate **Token Burn** dashboard, owned exclusively by the native launchd ingest pipeline (`run-all.sh` → `inject-dashboard.mjs`). Leave it alone.
- **Identity guard:** before editing, confirm the target file contains the marker `<h1>Claude Token Command Center</h1>`. If it does NOT, STOP immediately and send the alert (below) — do not write, or you may clobber the wrong dashboard.

## What to update (targeted Edits only — never rewrite the whole file)
The Command Center reads these embedded constants; update them in place:
1. `const USAGE_SUMMARY = {...}` — LIVE plan limits scraped from claude.ai/settings/usage (these are NOT token counts):
   - Open `https://claude.ai/settings/usage` via Chrome MCP, wait ~3s, read page text. If not logged in, screenshot and STOP → alert.
   - Extract: plan name, current session % + reset, weekly all-models % + reset, Sonnet-only % + reset, extra usage $ spent / $ limit, current balance, billing period. Build USAGE_SUMMARY with those fields and a `lastFetched` stamp (America/Los_Angeles).
2. `const DAILY_BURN = [...]` — EXACT per-day tokens, **Claude sources only (Cowork + Claude Code + Chat-est — NO Codex)**. Paste the full contents of `/Users/edmatibag/Documents/Claude/Projects/Token Burn Dashboard/daily-burn.json` as the JS array. (The dashboard's SOURCES list already excludes Codex; do not add a Codex column. Totals are computed Claude-only.)
3. `const SESSIONS = [...]` — paste `/Users/edmatibag/Documents/Claude/Projects/Token Burn Dashboard/sessions.json` if present (per-session rows). Skip if missing.
4. `const LAST_UPDATED = "..."` — today's date/time, America/Los_Angeles.

Skills usage and MCP-connector cards are computed client-side by the dashboard's own JS from SESSIONS — no extra edits needed.

## Data freshness note
The exact token figures come from the native ingesters (launchd `com.ed.tokenburn.ingest`, ~4:55 PM). You only OVERLAY live plan limits + re-paste the latest daily-burn.json/sessions.json into the Command Center file. You do NOT run the ingesters (the sandbox can't read the raw logs).

## Validate before finishing
- Reconcile each daily-burn row: `total == cowork_tokens + claude_code_tokens + claude_chat_est + codex_tokens` (cache excluded). If any row fails, STOP → alert (do not publish).
- Confirm the file still contains `<h1>Claude Token Command Center</h1>` after your edits.

## Completion summary
Report: plan limits (session %, weekly %, extra $/$, balance), exact total tokens (Claude sources, cache excluded) with Cowork/Claude Code split, top skills + top connector, and the dashboard path.

## Alert protocol (email Ed, then STOP — do not retry)
Trigger if: not logged into claude.ai, the identity marker is missing, reconciliation fails, a required MCP is unavailable, or the run stalls.
- Gmail: navigate to https://mail.google.com/mail/u/0/#compose, To: edmatibag9@gmail.com, Subject: ⚠️ Claude Token Command Center — [Category]: [one-line issue], Body: task name, date/time, category, issue (2-3 sentences), steps completed, action needed. Send (Cmd+Enter).
- Fallback if Chrome unavailable: create an Apple Note titled "⚠️ TASK ALERT: Claude Token Command Center — [date]" with the same body.
After sending, STOP.
