---
name: gws
description: >
  Google Workspace CLI for immediate, interactive operations on Gmail, Drive, Sheets,
  Calendar, Docs, Tasks, Contacts, and 25+ Google APIs. Use for any "do it now" Google
  task: read/send email, check calendar, upload to Drive, edit Sheets, manage contacts.
  NOT for scheduled automations (use /clasp). Triggers: "check my email", "calendar today",
  "upload to Drive", "send email", "agenda", "inbox", "contatti", any Google service interaction.
argument-hint: "<what-to-do> (e.g. 'check my calendar for today')"
allowed-tools: Bash(gws:*), Read, Write, Edit, Glob, Grep
---

# Google Workspace CLI (gws)

Interact with Google Workspace services directly from the terminal. gws dynamically
discovers all Google Workspace APIs and exposes them as a unified CLI.

## Dynamic Context

- gws installed: !`which gws 2>/dev/null && gws --version 2>/dev/null || echo "NOT INSTALLED"`
- Auth status: !`gws auth status 2>&1 | python3 -c "import sys,json; d=json.load(sys.stdin); print(f'Method: {d.get(\"auth_method\",\"none\")}  Account: {d.get(\"account\",\"unknown\")}')" 2>/dev/null || echo "NOT AUTHENTICATED"`
- Accounts: !`gws auth list 2>&1 | python3 -c "import sys,json; d=json.load(sys.stdin); accs=d.get('accounts',[]); print(', '.join(accs) if accs else 'No accounts')" 2>/dev/null || echo "Unknown"`
- Default account: !`gws auth list 2>&1 | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('default','none'))" 2>/dev/null || echo "Unknown"`

## Steps

### 0. Preflight Checks

1. **Check if gws is installed.** If not:
   ```bash
   npm install -g @googleworkspace/cli
   ```

2. **Check auth status.** If `auth_method` is `none`:
   - Tell the user to run `! gws auth login --account their@email.com`
   - If they need a GCP project first: `! gws auth setup`
   - **STOP and wait** — do not proceed until auth is confirmed

3. **If multiple accounts**, check which is the default and confirm with the user if needed.

### 1. Understand the Request

Identify:
- **Which Google service?** Gmail, Drive, Sheets, Calendar, Docs, Tasks, Chat, Forms, etc.
- **Read or write?** Read-only operations are safe; write operations need user confirmation
- **Helper available?** Check if a `+helper` command exists (faster and simpler)
- **API params needed?** What IDs, ranges, queries, etc. are required

### 2. Execute

Use the appropriate gws command. Prefer **helper commands** when available — they're simpler and handle encoding/formatting automatically.

**Always add `--format table`** when showing results to the user — it produces readable output instead of raw JSON. Only use `--format json` when you need to process the data programmatically.

**IMPORTANT SAFETY RULES:**
- **Always use `--dry-run` first** for write operations (send email, create event, delete file) and show the user what will happen
- **Never send emails, delete files, or modify data without explicit user confirmation**
- **For read-only operations** (list, get, search, triage), execute directly
- **Never output secrets** (API keys, tokens) directly

---

## Complete Service Reference

### Gmail

**Helpers:**
```bash
# Send an email (supports --cc, --bcc, --html, --from, -a/--attach)
gws gmail +send --to alice@example.com --subject 'Hello' --body 'Hi Alice!'
gws gmail +send --to alice@example.com --subject 'Report' --body 'See attached' -a report.pdf
gws gmail +send --to alice@example.com --subject 'FYI' --body 'Hi' --cc bob@co.com --bcc eve@co.com
gws gmail +send --to alice@example.com --subject 'News' --body '<b>Bold</b> text' --html

# Read a specific message (extracts body, handles base64/multipart)
gws gmail +read --id MSG_ID
gws gmail +read --id MSG_ID --headers        # Include From, To, Subject, Date
gws gmail +read --id MSG_ID --html           # Return HTML instead of plain text

# Reply to a message (handles threading, quoting automatically)
gws gmail +reply --message-id MSG_ID --body 'Thanks for the update!'
gws gmail +reply --message-id MSG_ID --body 'Looping in Carol' --cc carol@example.com
gws gmail +reply --message-id MSG_ID --body '<b>Bold reply</b>' --html
gws gmail +reply --message-id MSG_ID --body 'Updated version' -a updated.docx

# Reply all (same flags as +reply, plus --remove to exclude recipients)
gws gmail +reply-all --message-id MSG_ID --body 'Sounds good to me!'
gws gmail +reply-all --message-id MSG_ID --body 'Updated' --remove bob@example.com

# Forward a message
gws gmail +forward --message-id MSG_ID --to colleague@company.com
gws gmail +forward --message-id MSG_ID --to dave@co.com --body 'FYI see below'
gws gmail +forward --message-id MSG_ID --to dave@co.com -a extra-notes.pdf

# Triage inbox — show unread summary
gws gmail +triage
gws gmail +triage --max 5 --query 'from:boss'
gws gmail +triage --labels                   # Include label names

# Watch for new emails (streaming NDJSON, requires GCP project for Pub/Sub)
gws gmail +watch --project my-gcp-project
gws gmail +watch --project my-project --once --cleanup
```

**Raw API:**
```bash
# List/search messages
gws gmail users messages list --params '{"userId": "me", "maxResults": 10}'
gws gmail users messages list --params '{"userId": "me", "q": "from:boss@company.com is:unread"}'

# Get a specific message
gws gmail users messages get --params '{"userId": "me", "id": "MSG_ID"}'

# List labels
gws gmail users labels list --params '{"userId": "me"}'

# Get user profile
gws gmail users getProfile --params '{"userId": "me"}'
```

### Google Drive

**Helpers:**
```bash
# Upload a file (MIME type detected automatically)
gws drive +upload ./report.pdf
gws drive +upload ./report.pdf --parent FOLDER_ID --name 'Q4 Report.pdf'
```

**Raw API:**
```bash
# List files
gws drive files list --params '{"pageSize": 10}'

# Search files
gws drive files list --params '{"q": "name contains '\''report'\'' and mimeType = '\''application/pdf'\''", "pageSize": 20}'

# Get file metadata
gws drive files get --params '{"fileId": "FILE_ID"}'

# Download file
gws drive files get --params '{"fileId": "FILE_ID", "alt": "media"}' --output ./downloaded-file.pdf

# Create a folder
gws drive files create --json '{"name": "New Folder", "mimeType": "application/vnd.google-apps.folder"}'

# Move a file (add to new parent, remove from old)
gws drive files update --params '{"fileId": "FILE_ID", "addParents": "NEW_FOLDER_ID", "removeParents": "OLD_FOLDER_ID"}'

# Share a file
gws drive permissions create --params '{"fileId": "FILE_ID"}' --json '{"role": "reader", "type": "user", "emailAddress": "user@example.com"}'

# List files in a specific folder
gws drive files list --params '{"q": "'\''FOLDER_ID'\'' in parents", "pageSize": 50}'

# Export Google Doc as PDF
gws drive files export --params '{"fileId": "FILE_ID", "mimeType": "application/pdf"}' --output ./doc.pdf
```

### Google Sheets

**Helpers:**
```bash
# Read values from a range
gws sheets +read --spreadsheet SPREADSHEET_ID --range 'Sheet1!A1:D10'

# Read entire sheet
gws sheets +read --spreadsheet SPREADSHEET_ID --range Sheet1

# Append a row
gws sheets +append --spreadsheet SPREADSHEET_ID --values 'Alice,100,true'

# Append multiple rows
gws sheets +append --spreadsheet SPREADSHEET_ID --json-values '[["Alice",100],["Bob",200]]'
```

**Raw API:**
```bash
# Get spreadsheet metadata
gws sheets spreadsheets get --params '{"spreadsheetId": "ID"}'

# Update values
gws sheets spreadsheets values update --params '{"spreadsheetId": "ID", "range": "Sheet1!A1", "valueInputOption": "USER_ENTERED"}' --json '{"values": [["Name", "Score"], ["Alice", 95]]}'

# Batch get multiple ranges
gws sheets spreadsheets values batchGet --params '{"spreadsheetId": "ID", "ranges": ["Sheet1!A1:B5", "Sheet2!A1:C3"]}'

# Create a new spreadsheet
gws sheets spreadsheets create --json '{"properties": {"title": "My Sheet"}}'
```

### Google Calendar

**Helpers:**
```bash
# Show agenda (uses account timezone by default)
gws calendar +agenda                         # upcoming events
gws calendar +agenda --today                 # today only
gws calendar +agenda --tomorrow              # tomorrow only
gws calendar +agenda --week                  # this week
gws calendar +agenda --days 3                # next 3 days
gws calendar +agenda --calendar 'Work'       # specific calendar

# Create an event
gws calendar +insert --summary 'Team Standup' \
  --start '2026-03-24T09:00:00+01:00' \
  --end '2026-03-24T09:30:00+01:00' \
  --attendee alice@example.com \
  --location 'Room 3' \
  --description 'Weekly sync'
# Use --calendar ID to target a specific calendar (default: primary)
```

**Raw API:**
```bash
# List upcoming events
gws calendar events list --params '{"calendarId": "primary", "timeMin": "2026-03-23T00:00:00Z", "maxResults": 10, "singleEvents": true, "orderBy": "startTime"}'

# Get a specific event
gws calendar events get --params '{"calendarId": "primary", "eventId": "EVENT_ID"}'

# Delete an event
gws calendar events delete --params '{"calendarId": "primary", "eventId": "EVENT_ID"}'

# Check free/busy
gws calendar freebusy query --json '{"timeMin": "2026-03-24T08:00:00Z", "timeMax": "2026-03-24T18:00:00Z", "items": [{"id": "primary"}]}'

# List calendars
gws calendar calendarList list

# Quick add (natural language)
gws calendar events quickAdd --params '{"calendarId": "primary", "text": "Lunch with Alice tomorrow at noon"}'
```

### Google Docs

**Helpers:**
```bash
# Append text to a document
gws docs +write --document DOC_ID --text 'New paragraph here'
```

**Raw API:**
```bash
# Get document content
gws docs documents get --params '{"documentId": "DOC_ID"}'

# Batch update (insert, delete, format)
gws docs documents batchUpdate --params '{"documentId": "DOC_ID"}' --json '{"requests": [{"insertText": {"location": {"index": 1}, "text": "Hello World\n"}}]}'
```

### Google Tasks

```bash
# List task lists
gws tasks tasklists list

# List tasks in a task list
gws tasks tasks list --params '{"tasklist": "@default"}'

# Create a task
gws tasks tasks insert --params '{"tasklist": "@default"}' --json '{"title": "Review PR #42", "notes": "Check the auth changes", "due": "2026-03-25T00:00:00Z"}'

# Complete a task
gws tasks tasks patch --params '{"tasklist": "@default", "task": "TASK_ID"}' --json '{"status": "completed"}'
```

### Google Chat

**Helpers:**
```bash
# Send a message to a space
gws chat +send --space spaces/SPACE_ID --text 'Hello team!'
```

**Raw API:**
```bash
# List spaces
gws chat spaces list

# List messages in a space
gws chat spaces messages list --params '{"parent": "spaces/SPACE_ID"}'
```

### Google Forms

```bash
# Get a form
gws forms forms get --params '{"formId": "FORM_ID"}'

# List form responses
gws forms forms responses list --params '{"formId": "FORM_ID"}'
```

### Google Slides

```bash
# Get a presentation
gws slides presentations get --params '{"presentationId": "PRES_ID"}'

# Create a presentation
gws slides presentations create --json '{"title": "Q4 Review"}'
```

### Contacts (People API)

```bash
# List contacts
gws people people connections list --params '{"resourceName": "people/me", "personFields": "names,emailAddresses,phoneNumbers", "pageSize": 50}'

# Search contacts
gws people people searchContacts --params '{"query": "Alice", "readMask": "names,emailAddresses"}'
```

### Apps Script

**Helpers:**
```bash
# Push local files to an Apps Script project (replaces all remote files)
gws script +push --script SCRIPT_ID --dir ./src
```

**Raw API:**
```bash
# Get project content
gws apps-script projects getContent --params '{"scriptId": "SCRIPT_ID"}'

# Run a function
gws apps-script scripts run --params '{"scriptId": "SCRIPT_ID"}' --json '{"function": "myFunction"}'
```

### Events (Workspace Events API)

**Helpers:**
```bash
# Subscribe to Workspace events (streaming NDJSON, requires GCP project)
gws events +subscribe --target '//chat.googleapis.com/spaces/SPACE' \
  --event-types 'google.workspace.chat.message.v1.created' --project my-project

# Renew/reactivate subscriptions
gws events +renew --name subscriptions/SUB_ID
gws events +renew --all --within 2d
```

### Model Armor (Content Safety)

**Helpers for screening AI prompts/responses through safety templates:**
```bash
# Sanitize a user prompt
gws modelarmor +sanitize-prompt --template projects/P/locations/L/templates/T --text 'user input'

# Sanitize a model response
gws modelarmor +sanitize-response --template projects/P/locations/L/templates/T --text 'model output'

# Create a safety template
gws modelarmor +create-template --project P --location us-central1 --template-id my-tmpl --preset jailbreak
```

Any gws command can also screen its response through Model Armor using the global `--sanitize` flag:
```bash
gws gmail +triage --sanitize projects/P/locations/L/templates/T
```

### Additional Services

These services are available via raw API (`gws <service> <resource> <method>`). Use `gws schema` to discover parameters:

| Service | Alias | What it does |
|---------|-------|-------------|
| `google-keep` | `keep` | Notes and lists |
| `meet` | — | Conference records and meeting details |
| `admin` | `directory` | Users, groups, devices, org units |
| `admin-reports` | `reports` | Audit logs, usage reports |
| `vault` | — | eDiscovery matters and holds |
| `classroom` | — | Courses, invitations, user profiles |
| `cloudidentity` | — | Identity, devices, groups, SSO profiles |
| `alertcenter` | — | Security alerts and notifications |
| `groupssettings` | — | Group access settings |
| `licensing` | — | License assignments |
| `reseller` | — | Customer and subscription management |

---

## Cross-Service Workflows

Pre-built multi-service operations:

```bash
# Standup report: today's meetings + open tasks
gws workflow +standup-report

# Meeting prep: next meeting details, attendees, linked docs
gws workflow +meeting-prep

# Convert email to task
gws workflow +email-to-task --message-id MSG_ID

# Weekly digest: this week's meetings + unread count
gws workflow +weekly-digest

# Announce a Drive file in Chat
gws workflow +file-announce --file-id FILE_ID --space spaces/SPACE_ID
```

---

## API Discovery (Schema)

gws can discover any API schema dynamically:

```bash
# Get method schema (shows all parameters)
gws schema drive.files.list
gws schema gmail.users.messages.list
gws schema sheets.spreadsheets.values.update

# Resolve all $ref references
gws schema calendar.events.insert --resolve-refs
```

Use this when you need to find the exact parameters for an API call. The schema
shows required/optional params, request body structure, and response format.

---

## Output Formats

All commands support multiple output formats:

```bash
gws calendar +agenda --format table    # Human-readable table
gws calendar +agenda --format json     # JSON (default)
gws calendar +agenda --format yaml     # YAML
gws calendar +agenda --format csv      # CSV (for spreadsheet import)
```

Use `--format table` when showing results to the user.
Use `--format json` when processing data programmatically.

---

## Pagination

For large result sets:

```bash
# Auto-paginate through all results (NDJSON — one JSON per page)
gws drive files list --params '{"pageSize": 100}' --page-all

# Limit to N pages
gws drive files list --params '{"pageSize": 100}' --page-all --page-limit 5

# Add delay between pages (rate limiting)
gws drive files list --params '{"pageSize": 100}' --page-all --page-delay 200
```

---

## Multi-Account Support

```bash
# List accounts
gws auth list

# Set default account
gws auth default --account user@example.com

# Use a specific account for one command
GOOGLE_WORKSPACE_CLI_ACCOUNT=other@example.com gws gmail +triage
```

---

## Common Patterns

### Check email and summarize
```bash
gws gmail +triage --format table
```

### Find a file and share it
```bash
# Search
gws drive files list --params '{"q": "name contains '\''budget'\''", "pageSize": 5}' --format table
# Share (after getting FILE_ID)
gws drive permissions create --params '{"fileId": "FILE_ID"}' --json '{"role": "reader", "type": "user", "emailAddress": "colleague@company.com"}'
```

### Read sheet data and send summary email
```bash
# Read data
gws sheets +read --spreadsheet ID --range 'Sheet1!A1:D10' --format table
# Send summary (confirm with user first!)
gws gmail +send --to boss@company.com --subject 'Weekly Summary' --body 'Here are the numbers...'
```

### Morning briefing
```bash
gws workflow +standup-report --format table
gws gmail +triage --format table
```

---

## MCP Server Mode

gws can also run as an MCP server, giving Claude direct tool access to Google APIs:

```bash
# Start MCP with selected services (full mode — one tool per API method)
gws mcp -s drive,gmail,calendar,sheets

# Compact mode — one tool per service + discover tool
gws mcp -s drive,gmail,calendar --tool-mode compact

# Include workflow helpers and service helpers as tools
gws mcp -s drive,gmail,calendar -w -e

# Start with all services
gws mcp -s all
```

To configure in Claude Code settings, add to `mcpServers`:
```json
{
  "gws": {
    "command": "gws",
    "args": ["mcp", "-s", "drive,gmail,calendar,sheets,docs,tasks", "-w", "-e"]
  }
}
```

Note: MCP mode gives Claude direct API access without needing Bash commands.
The skill approach (this file) uses Bash commands instead, which gives more
visibility and control over what's being executed.

---

## Shell Gotchas

**Sheets ranges with `!`:** zsh interprets `!` as history expansion. Use double quotes for ranges:
```bash
# CORRECT (zsh-safe)
gws sheets +read --spreadsheet ID --range "Sheet1!A1:D10"

# MAY FAIL in zsh — ! triggers history expansion
gws sheets +read --spreadsheet ID --range 'Sheet1!A1:D10'
```

**JSON in flags:** Use single quotes around `--params` and `--json` to preserve inner double quotes:
```bash
gws drive files list --params '{"pageSize": 10}'
```

---

## Scope Management

Unverified (testing mode) apps are limited to ~25 OAuth scopes. If you hit this limit:

```bash
# Login with specific scopes only
gws auth login --scopes drive,gmail,sheets

# Or use service-specific flag
gws auth login -s drive,gmail,calendar
```

---

## Headless / CI Authentication

```bash
# On a machine with browser — export credentials
gws auth export --unmasked > credentials.json

# On headless machine — use exported credentials
export GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE=/path/to/credentials.json
gws gmail +triage
```

---

## Exit Codes

| Code | Meaning |
|------|---------|
| 0 | Success |
| 1 | API error (Google 4xx/5xx response) |
| 2 | Auth error (credentials missing/expired) |
| 3 | Validation error (bad args, unknown service) |
| 4 | Discovery error (API schema fetch failed) |
| 5 | Internal error (unexpected failure) |

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `GOOGLE_WORKSPACE_CLI_TOKEN` | Pre-obtained OAuth2 access token (highest priority) |
| `GOOGLE_WORKSPACE_CLI_CREDENTIALS_FILE` | Path to OAuth/service account JSON |
| `GOOGLE_WORKSPACE_CLI_CLIENT_ID` | OAuth client ID (alt to client_secret.json) |
| `GOOGLE_WORKSPACE_CLI_CLIENT_SECRET` | OAuth client secret |
| `GOOGLE_WORKSPACE_CLI_ACCOUNT` | Default account email for multi-account |
| `GOOGLE_WORKSPACE_CLI_CONFIG_DIR` | Override config dir (default: `~/.config/gws`) |
| `GOOGLE_WORKSPACE_CLI_LOG` | Stderr log level (e.g. `gws=debug`) |

Variables also load from `.env` file if present in the working directory.

---

## Official Persona Recipes

gws ships with pre-built persona workflows that combine multiple services. These are
useful patterns to follow when the user's request matches a persona:

| Persona | Services Used | Key Workflow |
|---------|--------------|--------------|
| **Exec Assistant** | Gmail, Calendar, Drive, Chat | Morning standup -> inbox triage -> meeting prep -> schedule management |
| **Project Manager** | Sheets, Chat, Calendar, Tasks | Task tracking in Sheets -> status updates in Chat -> calendar blocking |
| **IT Admin** | Admin, Reports, Licensing | User management -> audit logs -> permission reviews |
| **Sales Ops** | Gmail, Sheets, Docs, Drive | CRM updates in Sheets -> proposal generation in Docs -> email follow-ups |
| **Content Creator** | Docs, Drive, Slides, Gmail | Draft in Docs -> organize in Drive -> share via email |
| **Team Lead** | Calendar, Chat, Tasks, Gmail | Weekly planning -> team updates in Chat -> task assignment |
| **Researcher** | Drive, Docs, Sheets, Gmail | Collect sources in Drive -> notes in Docs -> data in Sheets |
| **HR Coordinator** | Admin, Calendar, Gmail, Docs | Onboarding users -> scheduling interviews -> offer letter generation |
| **Customer Support** | Gmail, Sheets, Chat, Tasks | Email triage -> log in Sheets -> escalate in Chat -> track in Tasks |
| **Event Coordinator** | Calendar, Gmail, Sheets, Forms | Schedule events -> send invites -> track RSVPs in Sheets |

When a user's request aligns with one of these patterns, follow the workflow sequence.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "auth_method: none" | Run `gws auth login --account your@email.com` |
| "This app isn't verified" | Click Advanced -> Go to [project name] (unsafe) -> Allow |
| Wrong account used | Check `gws auth list`, set default with `gws auth default --account ...` |
| API not enabled | gws prints enable URL in stderr — click link, wait 10s, retry |
| "Access blocked" on login | Add account to OAuth consent screen test users |
| `redirect_uri_mismatch` | OAuth client must be type "Desktop app" |
| "Permission denied" on admin APIs | Need Workspace admin privileges |
| Rate limited | Add `--page-delay` for pagination, reduce `pageSize` |
| "Invalid grant" | Token expired — run `gws auth login` again |
| `gcloud` not found for setup | Install gcloud, or set up OAuth manually in Cloud Console |

## Important Reminders

- **Always `--dry-run` before write operations** — show the user what will happen
- **Never send emails, delete files, or modify calendar without explicit confirmation**
- **Never output secrets** (API keys, tokens) directly — use `gws auth export` instead
- **Use `--format table`** when showing results to the user for readability
- **Use helpers** (`+send`, `+read`, `+reply`, `+forward`, `+triage`, `+append`, etc.) when available — simpler and safer
- **Use `gws schema`** to discover exact API parameters when unsure
- **gws is for NOW, clasp is for LATER** — immediate actions vs persistent automations
- **Single-quote JSON**, double-quote shell strings containing `!` (Sheets ranges)
- For bugs or feature requests: https://github.com/googleworkspace/cli/issues
