# Changelog

All notable changes to the Claude Token Command Center are documented here.
Format follows [Keep a Changelog](https://keepachangelog.com/); dates are America/Los_Angeles.
The real `claude-token-dashboard.html` and the source data files are gitignored as real usage data and
are never committed — only the sanitized `*.sample.html` and the code/docs are tracked.

## [2026-06-24] — Slack alerting + native-ingest freshness gate

### Added
- **Slack success alert (`#token-dashboard-alerts`, ID `C0BD8HUUTUG`).** On a fully clean run the daily
  refresh now posts one ✅ with the daily total, Cowork/Claude-Code split, session/weekly %, and a
  `file://` link to `token-burn-dashboard.html`. Success is Slack-only — no email or Apple Note.
- **Native-ingest freshness gate.** Before reporting success the task verifies the Token Burn ingest
  actually ran today: the latest `daily-burn.json` row must be today AND `token-burn-dashboard.html` must
  carry a `<span id="last-run">` stamped today. If either is stale it is treated as a `stale-ingest`
  failure and routed to the failure chain instead of posting ✅.

### Changed
- **Failure alert is now a tiered, independent chain:** **(1)** email Ed → **(2)** Slack ⚠️ to
  `#token-dashboard-alerts` → **(3)** Apple Note **only if both #1 and #2 fail to deliver**. Each send is
  independent (one failing never blocks the next), and the last-resort Apple Note now records *why* email
  and Slack each failed. Previously the protocol was email with an Apple Note fallback only when Chrome was
  unavailable.
- `Scheduled/SKILL.md`, `README.md`, and `AGENTS.md` updated to document the success/failure split and the
  freshness gate; `AGENTS.md` and `README.md` now cross-reference this changelog.

### Notes
- The `<span id="last-run">` stamp this task reads as its freshness gate is produced by the **Token Burn**
  project's `inject-dashboard.mjs` (separate repo), added in that project's 2026-06-24 entry.
- This task still writes only `claude-token-dashboard.html` and never touches `token-burn-dashboard.html`;
  the identity guard (`<h1>Claude Token Command Center</h1>`) is unchanged.
