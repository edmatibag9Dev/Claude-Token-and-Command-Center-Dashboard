# CONTRIBUTING.md

Standards for all commits, README updates, and GitHub pushes in this repository.

---

## Commit Message Format

### feat and fix commits — required format

```
feat: <subject line — imperative, max 72 chars>

- <bullet: what changed and why>
- <bullet: what was affected or replaced>
- <bullet: any constraint or caveat to note>
```

- Subject line must be imperative ("Add skill cards" not "Added skill cards")
- Body requires **at least 3 bullets**
- No one-liner commits for `feat` or `fix` types

### Other commit types

| Type | Use for | Body required? |
|---|---|---|
| `feat` | New features or sections | Yes — 3+ bullets |
| `fix` | Bug fixes or corrections | Yes — 3+ bullets |
| `data` | Data/content updates (scheduled refresh) | Yes — 3+ bullets |
| `docs` | README or documentation only | No |
| `chore` | Config, cleanup, non-functional | No |
| `style` | CSS/visual changes only | No |

### Example

```
feat: add Command Center skills and connector panels

- Added KNOWN_SKILLS config with 14 skill entries and keyword detection
- Added KNOWN_CONNECTORS config with 7 connector entries
- Added renderCommandCenter() that derives active/inactive status from sessions array
- Skills chart uses green/blue/grey for active/recent/inactive status
- Connector grid shows session count per connector with brand colors
```

---

## README Requirements

Update `README.md` on every `feat`, `fix`, or `data` commit. The README must always contain all 9 sections:

1. **Project title and one-line description**
2. **Overview** — problem solved and why it exists
3. **Features** — bulleted list of capabilities
4. **How It Works** — architecture or data flow
5. **File Structure** — repo tree with descriptions
6. **Setup** — steps to get running
7. **Configuration** — how to customize (skills, connectors, etc.)
8. **Scheduled Automation** — how the daily task works
9. **Contributing / License**

Minimum 400 words total. Use headers (`##`) for each section.

---

## GitHub Push Instructions (Contents API)

This repo is pushed via the GitHub Contents API, not git CLI.

### New file
```
PUT https://api.github.com/repos/edmatibag9Dev/Claude-Token-and-Command-Center-Dashboard/contents/{path}
Authorization: token <TOKEN>
Content-Type: application/json

{
  "message": "feat: <subject>\n\n- bullet\n- bullet\n- bullet",
  "content": "<base64-encoded file content>",
  "branch": "main"
}
```

### Update existing file
First retrieve the current `sha`:
```
GET https://api.github.com/repos/edmatibag9Dev/Claude-Token-and-Command-Center-Dashboard/contents/{path}
```
Then push with `sha` included:
```json
{
  "message": "data: refresh dashboard — <date>",
  "content": "<base64-encoded file content>",
  "sha": "<sha from GET response>",
  "branch": "main"
}
```

**Missing `sha` on an update = 409 Conflict. Always GET first.**

---

## What NOT to Commit

- GitHub tokens, API keys, or any credentials
- Full HTML file rewrites when only data changed — use targeted edits locally, commit the result
- Unrelated changes bundled into a `feat` or `fix` commit
