# TeamDesk Skill for Claude Code

A comprehensive [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skill for working with [TeamDesk](https://www.teamdesk.net) — a cloud-based database platform for building custom business applications.

## What's Included

| File | Description |
|------|-------------|
| `SKILL.md` | Main skill file — API patterns, formulas, workflows, n8n integration, and 20+ community patterns |
| `references/api-reference.md` | Complete REST API reference — all methods, parameters, response formats, error codes, hidden endpoints |
| `references/formula-reference.md` | Full formula function guide — 80+ functions with syntax, examples, and common patterns |

## Installation

### Claude Code CLI

```bash
# Clone into your skills directory
git clone https://github.com/Danielbluz/teamdesk-skill.git ~/.claude/skills/teamdesk
```

The skill will be automatically available in Claude Code sessions when working with TeamDesk databases.

### Manual

Copy the `SKILL.md` and `references/` folder into `~/.claude/skills/teamdesk/`.

## What It Covers

### REST API Integration
- Authentication (Bearer token)
- All CRUD methods: select, retrieve, create, update, upsert, delete
- Pagination, filtering, sorting
- Hidden endpoints: `exportview.aspx` (Excel/CSV export), `gateway.aspx` (auto-login), Email to Database
- Attachment upload/download
- Error handling and common pitfalls

### Formula Reference
- 80+ functions organized by category: text, date, time, numeric, rounding, aggregation, null handling, duration, timestamp, location, special
- Type conversion functions with culture/locale support
- Operators (logical, comparison, arithmetical) with type combination rules
- Summary columns and filters for table relationships

### Workflow Automation
- Trigger types: Record Change, Time-Dependent, Periodic
- Action types: Email, Record CRUD, Call URL, Navigate, Document Generation
- Webhooks with Response() extraction and iterators

### Community Patterns (from 100+ forum posts)
- **UI Enhancements**: Inline charts (QuickChart.io), progress bars (XHTML), popup forms (dbscript.js)
- **Workflow Patterns**: Status flip triggers, webhook notification-callback, auto-generate child records (RecordSet), master/detail clone
- **Data & API**: Excel export endpoint, custom auto-numbering, multi-record API batching, number-to-words auxiliary table
- **Client-Side**: JavaScript field access, cookies/localStorage, AI Data Analysis instructions
- **Document Generation**: CardText for numbers in words, dynamic checkboxes, subcategories via many-to-many
- **Import & Matching**: Multi-column matching, web-to-record auto-linking

### n8n Integration
- HTTP Request node setup for TeamDesk API
- Common workflow patterns (trigger → select → transform → upsert)
- Pagination loop in Code nodes

## Community Patterns Source

The "Community Patterns & Workarounds" section was curated from a systematic scan of the **100 most recent posts** on the [TeamDesk Community Forum](https://teamdesk.crmdesk.com/forum.aspx) (Feb 2026). Techniques were contributed by TeamDesk staff (Slava, Kirill) and power users (Scott Miller, Jorge Sola, Pierre, Calvin Peters, Nick Kemp) and verified against official documentation.

## License

MIT
