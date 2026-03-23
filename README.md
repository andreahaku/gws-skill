# gws-skill

A [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for interacting with Google Workspace directly from the terminal using the [gws CLI](https://github.com/googleworkspace/cli).

## What it does

This skill gives Claude Code real-time access to the entire Google Workspace ecosystem — Gmail, Drive, Sheets, Calendar, Docs, Tasks, Chat, Forms, Slides, Contacts, Admin, and more. Unlike scripting tools, gws executes operations immediately: read your email, upload files, create events, update spreadsheets — all from a single prompt.

### gws vs clasp

These skills are complementary:

| | gws | [clasp](https://github.com/andreahaku/clasp-skill) |
|---|---|---|
| **Purpose** | Immediate, interactive operations | Persistent automations on Google servers |
| **When** | Do something NOW | Build something that runs on a SCHEDULE |
| **Examples** | Read inbox, upload file, create event | Weekly email digest, webhook receiver, form generator |
| **Runs on** | Your terminal | Google's infrastructure |

## What's included

- **Complete service reference** for 15+ Google APIs with helper commands and raw API examples
- **Cross-service workflows** — standup reports, meeting prep, weekly digests, email-to-task
- **API discovery** via `gws schema` for finding exact parameters
- **Safety rules** — dry-run before writes, user confirmation before sends/deletes
- **Multi-account support** — switch between personal and work accounts
- **MCP server configuration** — optional direct API access mode for Claude Code
- **Output format control** — JSON, table, YAML, CSV

## Prerequisites

1. **Node.js** >= 20.0.0
2. **gws CLI** installed globally:
   ```bash
   npm install -g @googleworkspace/cli
   ```
3. **GCP project** with OAuth client configured:
   ```bash
   gws auth setup
   ```
4. **Authenticated** with Google:
   ```bash
   gws auth login --account your@email.com
   ```

## Installation

### Claude Code skill

```bash
# Clone the repo
git clone https://github.com/andreahaku/gws-skill.git ~/Development/Claude/gws-skill

# Symlink into Claude Code skills
ln -s ~/Development/Claude/gws-skill ~/.claude/skills/gws
```

The skill will appear as `/gws` in Claude Code.

## Usage

### Direct invocation

```
/gws check my email inbox
```

```
/gws what meetings do I have today?
```

```
/gws upload ./report.pdf to my Drive
```

```
/gws read the data from my Budget spreadsheet
```

### Proactive suggestions

The skill teaches Claude to recognize when gws is the right tool. For example, if you say:

> "Can you check if I have any unread emails from my boss?"

Claude will use gws to query your Gmail directly.

## Supported Google Services

| Service | Helpers | Key Operations |
|---------|---------|---------------|
| **Gmail** | `+send`, `+triage`, `+watch` | Send, search, read, label emails |
| **Drive** | `+upload` | Upload, download, search, share, organize files |
| **Sheets** | `+read`, `+append` | Read ranges, append rows, update values |
| **Calendar** | `+agenda`, `+insert` | List events, create events, check availability |
| **Docs** | `+write` | Read and append to documents |
| **Tasks** | — | Create, list, complete tasks |
| **Chat** | `+send` | Send messages to spaces |
| **Forms** | — | Read forms and responses |
| **Slides** | — | Read and create presentations |
| **People** | — | Search and manage contacts |
| **Admin** | — | User/group management, audit logs |

### Cross-Service Workflows

| Workflow | What it does |
|----------|-------------|
| `+standup-report` | Today's meetings + open tasks |
| `+meeting-prep` | Next meeting: agenda, attendees, linked docs |
| `+email-to-task` | Convert a Gmail message into a Google Task |
| `+weekly-digest` | This week's meetings + unread email count |
| `+file-announce` | Announce a Drive file in a Chat space |

## Optional: MCP Server Mode

gws can also run as an MCP server for direct API access:

```json
{
  "mcpServers": {
    "gws": {
      "command": "gws",
      "args": ["mcp", "-s", "drive,gmail,calendar,sheets,docs,tasks", "-w", "-e"]
    }
  }
}
```

## License

MIT
