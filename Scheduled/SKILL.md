---
name: token-dashboard-update
description: Refresh Ed's Claude Command Center dashboard daily at 4PM — token usage, skills activity, and connector usage
---

## Task: Refresh Claude Command Center from claude.ai/settings/usage

**Objective:** Pull real token usage data from claude.ai/settings/usage using the browser, then update the Command Center dashboard at `/Users/edmatibag/Claude/claude-token-dashboard.html`. The dashboard includes three sections: (1) token usage summary, (2) Command Center skills usage panel, and (3) MCP connector usage panel.

---

### Step 1 — Navigate to Claude Usage Page

Use the Chrome MCP tools (`mcp__Claude_in_Chrome__tabs_context_mcp` then `mcp__Claude_in_Chrome__navigate`) to open:
`https://claude.ai/settings/usage`

Wait 3 seconds for the page to fully load, then use `mcp__Claude_in_Chrome__get_page_text` to extract all text content from the page.

If not logged in, take a screenshot and stop — notify that manual login is required.

---

### Step 2 — Extract Usage Data

From the page text, extract:
- Total tokens used (current billing period)
- Messages sent (if shown)
- Any breakdown by model or session type if available
- The billing period dates (e.g. "Mar 15 – Apr 15")
- Usage percentage if shown (e.g. "42% of limit used")

Also use `mcp__Claude_in_Chrome__javascript_tool` to try extracting structured data if the page text is sparse:
```javascript
document.body.innerText
```

---

### Step 3 — Get Recent Session List

Use `mcp__session_info__list_sessions` with limit 50 to get all recent Cowork sessions.

For each session, categorize by type:
- **dispatch** → title contains "dispatch", "brain", "openbrain" (case-insensitive)
- **cowork** → everything else (Cowork is the default for all local sessions)
- **chat** → there will be no local chat sessions; this type comes from claude.ai aggregate only

Extract session names and dates. Estimate token usage per session using these averages based on session complexity:
- Simple sessions (short titles, admin tasks like "View file", "Fix error"): ~5,000 input / 1,500 output
- Medium sessions (reports, analysis, journal entries): ~25,000 input / 5,000 output
- Heavy sessions (multi-source scraping, long automation builds): ~80,000 input / 8,000 output

Classify each session as simple/medium/heavy based on its title.

---

### Step 4 — Build Real Sessions Array

Combine the claude.ai aggregate total with the session-level estimates. Create a sessions array with the 25 most recent sessions, formatted as:

```json
{
  "sessions": [
    {
      "name": "Session name",
      "type": "cowork|chat|dispatch",
      "date": "YYYY-MM-DD",
      "inputTokens": 25000,
      "outputTokens": 5000,
      "cacheTokens": 8000,
      "cost": 0.12
    }
  ]
}
```

Cost formula (claude-sonnet-4-6):
- cost = (inputTokens × 0.000003) + (outputTokens × 0.000015) + (cacheTokens × 0.0000003)

For dispatch sessions, use lower estimates: ~2,000 input / 600 output.

---

### Step 5 — Add a Real Usage Summary Block

Capture the raw claude.ai aggregate stats as a separate summary. You'll update this as a JS variable called `USAGE_SUMMARY`:

```javascript
const USAGE_SUMMARY = {
  billingPeriod: "Mar 15 – Apr 15",
  totalTokens: 1250000,
  usagePct: 42,
  messagesUsed: 380,
  lastFetched: "Apr 3, 2026 4:02 PM"
};
```

---

### Step 6 — Update the Dashboard File

Read `/Users/edmatibag/Claude/claude-token-dashboard.html`.

Make these targeted updates using the Edit tool. **Never overwrite the full file — only replace the specific blocks listed below.**

1. **Replace the `sessions: [...]` array inside `const REAL_DATA`** with the new sessions array from Step 4. Do not touch the `const REAL_DATA = {` wrapper line.

2. **Replace `const USAGE_SUMMARY = { ... }`** with the fresh values from Step 2 (billing period, usage %, messages used, lastFetched timestamp).

3. **Update the four `<div class="meta-item">` blocks** inside `<div class="header-meta">` to reflect:
   - Current billing period
   - Current session % and weekly % usage
   - Extra usage balance and spend
   - Last updated timestamp (today's date + "Automated Update ✅ Live from claude.ai")

4. **Update the three summary cards** (Current Session %, Weekly Usage %, Extra Usage Spending $) with fresh values.

5. **Update the `<div class="last-updated">` content** to today's date and the live data note.

> ⚠️ Do NOT touch `const KNOWN_SKILLS`, `const KNOWN_CONNECTORS`, or `renderCommandCenter()` — these are static Command Center config and auto-derive from the sessions array you updated in step 1.

Save the file.

---

### Step 7 — Confirm Completion

Output a summary:
- ✅ Sessions captured: [N]
- ✅ Billing period: [dates]
- ✅ Total tokens (claude.ai): [number]
- ✅ Estimated breakdown: Cowork [X%] / Dispatch [Y%] / Chat [Z%]
- ✅ Active skills detected: [list of skills with count > 0]
- ✅ Inactive skills: [list of skills with count = 0]
- ✅ Top connector: [connector name] ([N] sessions)
- ✅ Command Center updated at: /Users/edmatibag/Claude/claude-token-dashboard.html
- ⚠️ Note if claude.ai data was unavailable and estimates were used

---

### Constraints
- Never overwrite the full HTML file — use targeted Edit tool replacements only
- If claude.ai/settings/usage is inaccessible, use session_info estimates only and note it clearly in the dashboard header
- Keep the 25 most recent sessions maximum
- Ed's workspace: `/Users/edmatibag/Claude/`
- Dashboard file: `/Users/edmatibag/Claude/claude-token-dashboard.html`

---

## 🚨 Alert Protocol — Email Ed When Blocked or an Error Occurs

**Trigger this alert and stop the task immediately when any of the following occur:**

**Category 1 — System / MCP Error:**
- An MCP service or tool is unavailable, unreachable, or returns a persistent error
- A required connector, permission, or instruction is missing
- The task is stalled or taking excessively long and must be paused

**Category 2 — User Action Required:**
- Chrome browser is not open or cannot be accessed
- Login, authorization, or access approval is needed before proceeding (e.g. not logged into claude.ai)
- Any step requires Ed's direct interaction to continue

**How to send the alert:**

**Step 1 — Send via Gmail (preferred):**
1. Use `mcp__Claude_in_Chrome__navigate` to go to `https://mail.google.com/mail/u/0/#compose`
2. Fill the compose form using `mcp__Claude_in_Chrome__javascript_tool`:
   - **To:** edmatibag9@gmail.com
   - **Subject:** ⚠️ Claude Command Center — [Category]: [One-line issue]
   - **Body:**
     ```
     Scheduled Task: token-dashboard-update
     Date/Time: [current date and time]
     Alert Category: System Error  OR  User Action Required

     Issue:
     [2–3 sentences describing exactly what went wrong or what is needed to proceed]

     Steps completed before stopping:
     [Brief summary of what was successfully done before the error]

     Action needed from Ed:
     [Specific thing Ed needs to do to unblock the task]

     — Cowork Automated Alert
     ```
3. Send using `mcp__Claude_in_Chrome__shortcuts_execute` with Cmd+Enter, or use `mcp__Claude_in_Chrome__find` to click the Send button.

**Step 2 — Fallback if Chrome is unavailable:**
Use `mcp__Read_and_Write_Apple_Notes__add_note` with:
- **Title:** ⚠️ TASK ALERT: Claude Command Center — [Date]
- **Body:** Same content as the email body above

**After sending the alert:** Stop the task. Do not loop or retry indefinitely.