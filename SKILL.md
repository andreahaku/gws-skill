---
name: gws
description: >
  Interact with Google Workspace directly from the terminal using the gws CLI.
  This skill provides real-time access to Gmail, Google Drive, Sheets, Calendar,
  Docs, Slides, Forms, Tasks, Chat, Contacts, and 15+ more Google APIs — all
  through a single unified CLI. MUST be consulted for ANY task that requires
  reading, writing, or managing data in Google Workspace services RIGHT NOW
  (not scheduled automations — use /clasp for those).


  ALWAYS trigger when the user wants to:
  (1) Read, send, search, or triage Gmail (inbox summary, send email, find messages,
  convert email to task, draft responses),
  (2) Manage Google Drive files (upload, download, list, search, share, organize,
  move, copy, delete files and folders),
  (3) Read or write Google Sheets data (read cells, append rows, update values,
  create spreadsheets, bulk data operations),
  (4) Manage Google Calendar (list events, create events, check availability,
  meeting prep, agenda, free/busy lookup),
  (5) Create or edit Google Docs (append text, read document content),
  (6) Manage Google Tasks (create tasks, list tasks, mark complete, convert
  email to task),
  (7) Send messages to Google Chat spaces,
  (8) Read or manage Google Forms and responses,
  (9) Manage Google Slides presentations,
  (10) Look up or manage contacts via People API,
  (11) Run cross-service workflows (standup report, meeting prep, weekly digest,
  email-to-task conversion, file announcements),
  (12) Admin operations (user management, audit logs, licensing, groups),
  (13) Discover API schemas and capabilities for any Google service,
  (14) Any request that says "check my email", "what's on my calendar",
  "upload this to Drive", "add a row to my sheet", "send an email to X",
  "what meetings do I have", "create a calendar event", "read my inbox".


  Key differentiator vs /clasp: gws is for IMMEDIATE, INTERACTIVE operations
  (read my email now, upload this file now, check my calendar now). /clasp is
  for PERSISTENT AUTOMATIONS that run on Google's servers (weekly triggers,
  webhooks, form generators). If the user wants to DO something in Google
  Workspace right now, use /gws. If they want to BUILD something that runs
  on a schedule, use /clasp.
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

Use the appropriate gws command. Prefer **helper commands** (`+send`, `+read`, `+append`, etc.) when available — they're simpler and handle encoding/formatting automatically.

**IMPORTANT SAFETY RULES:**
- **Always use `--dry-run` first** for write operations (send email, create event, delete file) and show the user what will happen
- **Never send emails, delete files, or modify data without explicit user confirmation**
- **For read-only operations** (list, get, search, triage), execute directly

---

## Complete Service Reference

### Gmail

**Helpers:**
```bash
# Send an email
gws gmail +send --to alice@example.com --subject 'Hello' --body 'Hi Alice!'

# Reply to a message
gws gmail +reply --message-id MSG_ID --body 'Thanks for the update!'

# Reply all
gws gmail +reply-all --message-id MSG_ID --body 'Sounds good to me!'

# Forward a message
gws gmail +forward --message-id MSG_ID --to colleague@company.com

# Triage inbox — show unread summary
gws gmail +triage

# Read a specific message (extracts body, handles base64/multipart)
gws gmail +read --id MSG_ID
gws gmail +read --id MSG_ID --headers        # Include From, To, Subject, Date
gws gmail +read --id MSG_ID --html           # Return HTML instead of plain text

# Watch for new emails (streaming NDJSON)
gws gmail +watch
```

**Raw API:**
```bash
# List messages
gws gmail users messages list --params '{"userId": "me", "maxResults": 10}'

# Search messages
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
# Upload a file
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
# Show today's agenda (uses account timezone, cached 24h)
gws calendar +agenda

# Show agenda with specific timezone
gws calendar +agenda --timezone Europe/Rome

# Create an event
gws calendar +insert --summary 'Team Standup' \
  --start '2026-03-24T09:00:00+01:00' \
  --end '2026-03-24T09:30:00+01:00' \
  --attendee alice@example.com \
  --location 'Room 3'
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
gws script +push --script-id SCRIPT_ID --dir ./src
```

**Raw API:**
```bash
# List Apps Script projects
gws apps-script projects list

# Get project content
gws apps-script projects getContent --params '{"scriptId": "SCRIPT_ID"}'

# Run a function
gws apps-script scripts run --params '{"scriptId": "SCRIPT_ID"}' --json '{"function": "myFunction"}'
```

### Events (Workspace Events API)

**Helpers:**
```bash
# Subscribe to Workspace events (streaming NDJSON)
gws events +subscribe

# Renew/reactivate subscriptions
gws events +renew
```

### Google Keep

```bash
# List notes
gws keep notes list

# Get a note
gws keep notes get --params '{"name": "notes/NOTE_ID"}'
```

### Google Meet

```bash
# List conference records
gws meet conferenceRecords list

# Get meeting details
gws meet conferenceRecords get --params '{"name": "conferenceRecords/REC_ID"}'
```

### Admin (Workspace Admin)

```bash
# List users
gws admin users list --params '{"domain": "company.com", "maxResults": 50}'

# Get user info
gws admin users get --params '{"userKey": "user@company.com"}'

# List groups
gws admin groups list --params '{"domain": "company.com"}'

# Audit logs
gws admin-reports activities list --params '{"userKey": "all", "applicationName": "login"}'
```

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
# Send summary
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
# Start MCP with selected services
gws mcp -s drive,gmail,calendar,sheets

# Start with all services
gws mcp -s all

# Include workflow helpers
gws mcp -s drive,gmail,calendar -w -e
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

**Sheets ranges with `!`:** Bash/zsh interpret `!` as history expansion. Always use single quotes:
```bash
# CORRECT
gws sheets +read --spreadsheet ID --range 'Sheet1!A1:D10'

# WRONG — bash will try history expansion
gws sheets +read --spreadsheet ID --range "Sheet1!A1:D10"
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
| **Exec Assistant** | Gmail, Calendar, Drive, Chat | Morning standup → inbox triage → meeting prep → schedule management |
| **Project Manager** | Sheets, Chat, Calendar, Tasks | Task tracking in Sheets → status updates in Chat → calendar blocking |
| **IT Admin** | Admin, Reports, Licensing | User management → audit logs → permission reviews |
| **Sales Ops** | Gmail, Sheets, Docs, Drive | CRM updates in Sheets → proposal generation in Docs → email follow-ups |
| **Content Creator** | Docs, Drive, Slides, Gmail | Draft in Docs → organize in Drive → share via email |
| **Team Lead** | Calendar, Chat, Tasks, Gmail | Weekly planning → team updates in Chat → task assignment |
| **Researcher** | Drive, Docs, Sheets, Gmail | Collect sources in Drive → notes in Docs → data in Sheets |
| **HR Coordinator** | Admin, Calendar, Gmail, Docs | Onboarding users → scheduling interviews → offer letter generation |
| **Customer Support** | Gmail, Sheets, Chat, Tasks | Email triage → log in Sheets → escalate in Chat → track in Tasks |
| **Event Coordinator** | Calendar, Gmail, Sheets, Forms | Schedule events → send invites → track RSVPs in Sheets |

When a user's request aligns with one of these patterns, follow the workflow sequence.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "auth_method: none" | Run `gws auth login --account your@email.com` |
| "This app isn't verified" | Click Advanced → Go to [project name] (unsafe) → Allow |
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
- **Use helpers (`+send`, `+read`, `+append`, etc.)** when available — simpler and safer
- **Use `gws schema`** to discover exact API parameters when unsure
- **gws is for NOW, clasp is for LATER** — immediate actions vs persistent automations
- **Single-quote shell strings** containing `!` (Sheets ranges) or JSON
