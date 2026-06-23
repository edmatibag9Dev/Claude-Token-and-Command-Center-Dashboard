# Claude Token Command Center

A locally-hosted, self-updating dashboard for Claude **Cowork + Claude Code** usage: live plan limits,
exact per-day token burn by source, the heaviest sessions, which skills are active or idle, and which
MCP connectors are in use — all in a single self-contained HTML file (`claude-token-dashboard.html`)
that a Cowork scheduled task refreshes daily.

> **This is one of two dashboards. Don't collide them.**
> - **This repo** owns `claude-token-dashboard.html` (the Command Center). It is **Claude-sources-only**
>   (Cowork + Claude Code + Chat-est) — **no Codex**.
> - The separate [Token Burn Dashboard](https://github.com/edmatibag9Dev/Token-Burn-Dashboard-)
>   (`token-burn-dashboard.html`) is the heatmap/dotplot of burn across **all** sources, **including Codex**.
> - For ~30 days the two overwrote each other because both wrote `claude-token-dashboard.html`. Fixed
>   2026-06-22: one writer per file, each guarded by an identity marker (see below).

---

## Ownership & identity guard (why the dashboards stopped fighting)

- The daily refresh task writes **only** `claude-token-dashboard.html` and **must verify** the file
  contains `<h1>Claude Token Command Center</h1>` before editing. If the marker is missing, it stops and
  alerts instead of writing — so it can never clobber the Token Burn dashboard.
- It must **never** write `token-burn-dashboard.html` (owned by the Token Burn project's native launchd
  pipeline).
- Exact token numbers come from the Token Burn project's `daily-burn.json` (the shared source of truth);
  this task only re-pastes them and overlays live plan limits.

## Features

- **Live plan limits** — current-session %, weekly all-models %, Sonnet-only %, extra-usage $ and balance,
  scraped from `claude.ai/settings/usage`.
- **Exact token usage (Claude sources only)** — totals, daily burn by source, and source mix for
  Cowork + Claude Code + Chat-est. Codex is intentionally excluded (it lives on the Token Burn dashboard).
- **Sessions**, **Skills usage** (active / recent / inactive), and **MCP connector** cards — computed
  client-side from the embedded `SESSIONS` data.
- **Daily detail** table — exact per-day Claude-source tokens + cache.

## Files

```
├── claude-token-dashboard.html         # The Command Center — gitignored (contains real usage)
├── claude-token-dashboard.sample.html  # Committed reference layout with placeholder sample data
├── Scheduled/SKILL.md                  # The daily 5:02 PM refresh task (ownership-guarded)
├── .gitignore                          # keeps real-usage html/data out of git
├── AGENTS.md                           # AI agent instructions
├── CONTRIBUTING.md                     # commit + README standards
└── README.md                           # this file
```

## Data & Privacy

The real `claude-token-dashboard.html` embeds your actual session titles, so it is **gitignored** —
only the sanitized `*.sample.html` is committed. All data stays local; the only outbound call is the
daily scrape of `claude.ai/settings/usage`.

## Refresh

The Cowork scheduled task `claude-token-dashboard-update` runs daily at 5:02 PM PT: scrape plan limits →
re-paste `daily-burn.json` / `sessions.json` (Claude sources) → targeted edits to
`claude-token-dashboard.html` → completion summary, with an email/Apple-Notes alert if blocked.

---

_Last updated: 2026-06-22 — renamed to Command Center, Claude-sources-only (Codex removed), single-writer ownership + identity guard._
