---
name: teamdesk
description: TeamDesk online database platform expertise for API integrations, formula creation, workflow automation, and database design. Use when working with TeamDesk databases including REST API calls (select, create, update, upsert, delete), writing formulas, configuring webhooks, setting up triggers and actions, designing table relationships (one-to-many, many-to-many), or integrating TeamDesk with n8n workflows and other automation tools.
---

# TeamDesk Skill

TeamDesk is a cloud-based database platform for building custom business applications without coding. This skill covers API integration, formulas, automation, and database design.

## Quick Reference

### REST API Base URL
```
https://www.teamdesk.net/secure/api/v2/{database_id}/{table}/{method}.json
```

### Authentication (Required)
```
Authorization: Bearer {token}
```

- `database_id`: Found in URL after `/db/` (e.g., `https://www.teamdesk.net/secure/db/12345/...` → `12345`)
- `token`: 32-character authorization token from Setup > Database > Integration API
- `table`: Table name (URL-encoded if contains spaces)

> **Security note:** Always use the `Authorization: Bearer` header. Avoid embedding tokens in the URL, as URLs may be logged in server/proxy logs and browser history.

### API Methods Overview

| Method | HTTP | Purpose |
|--------|------|---------|
| select | GET | Query records from table or view |
| retrieve | GET | Get specific record by ID/key |
| create | POST | Create new records |
| update | POST | Update existing records |
| upsert | POST | Create or update (match by column) |
| delete | GET/POST | Delete records |
| describe | GET | Get table/column metadata |
| attachment | GET | Download file attachments |

### Common API Patterns

**Select with filter and pagination:**
```
GET /api/v2/{db_id}/{table}/select.json?filter=[Status]="Active"&column=Name&column=Email&top=100&skip=0&sort=Name//ASC
Authorization: Bearer {token}
```

**Upsert (create or update):**
```
POST /api/v2/{db_id}/{table}/upsert.json?match=External_ID
Authorization: Bearer {token}
Content-Type: application/json
[{"External_ID": "ABC123", "Name": "Updated Name", "Value": 100}]
```

**Select from View:**
```
GET /api/v2/{db_id}/{table}/{view}/select.json?top=500
Authorization: Bearer {token}
```

### Key API Notes

- Default limit: 500 records per request (use `skip` and `top` for pagination)
- Dates format: `YYYY-MM-DDTHH:MM:SS+00:00`
- Duration: Number of seconds
- Checkboxes: `true` or `false`
- Triggers execute by default on create/update/upsert (disable with `?triggers=false` if user has Manage Data privilege)

## Formula Quick Reference

### Syntax Basics
- Column reference: `[Column Name]`
- Variable: `Var[Variable Name]`
- User property: `User[Property Name]`
- Related column: `Related[Column Name]` (only in summary filters)

### Essential Functions

**Conditionals:**
```
If([Status]="Paid", "Complete", "Pending")
Case([Priority], "High", 1, "Medium", 2, 3)
```

**Date/Time:**
```
Today()                          // Current date
Now()                            // Current timestamp
Year([Date]) / Month([Date]) / Day([Date])
DateDiff("d", [Start], [End])    // Days between dates
[Date] + Days(30)                // Add duration
AdjustMonth([Date], 1)           // Add months
```

**Text:**
```
List(", ", [First], [Last])      // Join with separator
Left([Text], 5) / Right([Text], 5)
Contains([Text], "search")       // Returns checkbox
URLEncode([Text])
```

**Type Conversion:**
```
ToNumber("123.45")
ToText([Number])
ToDate("2024-01-15")
Days([Number]) / Hours([Number]) / Minutes([Number])
```

**Null Handling:**
```
IsNull([Field])
IfNull([Field], "default")
```

### Aggregation (Summary Columns)
```
Total([Amount])
Count([Id])
Average([Value])
Min([Date]) / Max([Date])
Concatenate(", ", [Name])
```

## Workflow Automation

### Trigger Types
1. **Record Change**: Fires on create/update/delete of specific columns
2. **Time-Dependent**: Fires relative to a date column (e.g., 3 days before Due Date)
3. **Periodic**: Fires on schedule (daily, weekly, monthly)

### Action Types
- Email Alert
- Record Create/Update/Delete
- Call URL (HTTP request to external API)
- Navigate
- Document Generation

### Webhooks (Receiving Data)
- Endpoint: `https://www.teamdesk.net/secure/db/{id}/hooks/{endpoint}`
- Supports JSON, XML, form-data
- Use Response() function to extract fields
- Iterators allow processing arrays (one record per item)

## Table Relationships

### One-to-Many
- Master table has Key column (usually Id)
- Detail table has Reference column pointing to master
- Creates Lookup columns (pull data from master) and Summary columns (aggregate from details)

### Many-to-Many
- Uses Link table with two reference columns
- Or use match conditions (e.g., Designer=Designer between Hours and Payments)
- Multi-reference column provides UI for selecting multiple related records

## n8n Integration Patterns

### HTTP Request Node Setup
```
Method: GET (select) or POST (create/update/upsert)
URL: https://www.teamdesk.net/secure/api/v2/{db_id}/{table}/select.json
Headers:
  Authorization: Bearer {token}
  Content-Type: application/json
```

### Common Workflow Pattern
1. **Trigger**: Webhook, Schedule, or another app
2. **TeamDesk Select**: Get existing records
3. **Process**: Transform data with Code/Set nodes
4. **TeamDesk Upsert**: Create or update records using match column

### Pagination Loop
```javascript
// In Code node for handling >500 records
let allRecords = [];
let skip = 0;
const top = 500;
let hasMore = true;

while (hasMore) {
  const response = await $http.get(url + `?top=${top}&skip=${skip}`);
  allRecords = allRecords.concat(response.data);
  hasMore = response.data.length === top;
  skip += top;
}
return allRecords;
```

## Reference Files

For detailed information, see:
- **references/api-reference.md**: Complete API method documentation, parameters, response formats, error codes
- **references/formula-reference.md**: Full function list organized by category with examples

## Community Patterns & Workarounds

*Curated from 100+ forum posts (Feb 2026 scan). Proven techniques shared by TeamDesk power users.*

### UI Enhancements

**Inline Charts with QuickChart.io (Formula-URL + XHTML)**
Embed live charts inside forms using a Formula-URL column:
```
"https://quickchart.io/chart?c=" & URLEncode("{type:'bar',data:{labels:['" & [Label1] & "','" & [Label2] & "'],datasets:[{data:[" & [Value1] & "," & [Value2] & "]}]}}")
```
Display via Formula-XHTML: `"<img src='" & [Chart URL] & "' width='400'/>"`. Customize with CSS in `dbstyles-V3.css`.

**Progress Bar in Forms (Formula-XHTML)**
```
"<div class='pb-container'><progress value='" & ToText([Percentage]) & "' max='100'></progress><span>" & Format([Percentage], "0") & "%</span></div>"
```
Style in `dbstyles-V3.css`:
```css
.pb-container progress { width: 200px; height: 20px; }
.pb-container progress::-webkit-progress-value { background: #4CAF50; }
```

**Pre-filtered View URLs in Dashboards**
Create clickable links that open a view with filters pre-applied:
```
URLRoot() & "/table/view?filter=" & URLEncode("[Status]=""Overdue""")
```
Useful for dashboard tiles showing counts that drill into filtered lists.

**Pop-up Forms via dbscript.js**
Open a quick-entry form in a popup window without leaving the current record:
```javascript
// In dbscript.js
$("#popup-btn").click(function(){
  window.open(URLRoot + "/table/new?field=value", "", "width=800,height=600");
});
```

### Workflow Patterns

**Status Flip Pattern (Conditional Trigger Execution)**
Use a checkbox or status column as a "trigger switch":
1. Create a checkbox column `Run Export` (default: unchecked)
2. Create a Record Change Trigger on `Run Export` with filter `[Run Export] = true`
3. Trigger action executes (e.g., Call URL, Email, Record Create)
4. Final action: Record Update to reset `Run Export` = false

This gives users an on-demand button for workflow execution, avoiding the need for manual triggers.

**Webhook Notification → Callback Pattern**
When receiving webhooks from external systems that send minimal data:
1. Webhook receives short notification (e.g., `{id: 123, event: "updated"}`)
2. Use `Response("id")` to extract the ID
3. Chain a Call URL action to fetch full data: `GET https://api.external.com/records/` & Response("id")
4. Process the full response with `Response("field.subfield")`

**Auto-generate N Child Records (RecordSet as Source)**
Create multiple child records automatically from a parent:
1. Parent table has a RecordSet column pointing to a "template" table
2. Record Create action uses the RecordSet column as its source
3. TeamDesk iterates automatically, creating one child per RecordSet item
4. Each child inherits values from the template + `ParentKey()` for the reference

**Master/Detail Clone (RecordSet + Workflow)**
Clone a parent record with all its children using workflows:
1. Record Create action creates the new parent (with modified fields)
2. Use RecordSet column (pointing to original children) as source for child creation
3. Workflow chain: Create Parent → Get New Parent Key → Create Children with `ParentKey()`

Note: Custom Actions are obsolete. Use RecordSet + workflow actions instead.

### Data & API Patterns

**Export View as Excel (Hidden Endpoint)**
```
URLRoot() & "/exportview.aspx/filename.xlsx?format=2&id=" & [View ID]
```
Formats: `0`=CSV, `2`=XLSX. Useful for scheduled report generation via Call URL or n8n.

**Email to Database (Native Feature)**
Setup > Tools > Email to Database creates a unique email address per table. Incoming emails become records with fields mapped from Subject, Body, From, Attachments. No external tools needed.

**Custom AutoNumbering (Per-Category Sequences)**
For sequences like "INV-2026-001" per client or category:
1. Create a Number column `Sequence` with Unique constraint
2. Formula-Number: `Max([Sequence]) + 1` (filtered by category via Summary)
3. Set via Record Create action or default value
4. Unique constraint prevents duplicates under concurrent access

**Multi-Record Single API Call (Array Formula + Summary)**
Batch data from multiple child records into one API call:
1. Child records each have a Formula-Text building their JSON fragment
2. Parent has Summary column: `Concatenate(",", [JSON Fragment])`
3. Call URL body uses: `"[" & [Concatenated JSON] & "]"`
4. Single API call sends all children's data

**Number-to-Words via Auxiliary Table**
More reliable than nested formulas for converting amounts to text:
1. Create helper table with 1,000 records (0-999), each with the word representation
2. Lookup column references the helper table by numeric value
3. Combine thousands + hundreds + units via formula
4. Works for any language (Portuguese, Spanish, English)

**Bulk File Download (Downloadyze Extension)**
Chrome extension [Downloadyze](https://chromewebstore.google.com/detail/downloadyze) can bulk-download all file attachments from a TeamDesk view. Useful for backup or migration.

### Client-Side Customization (dbscript.js / dbstyles-V3.css)

**JavaScript Field Value Access**
Read field values in dbscript.js using the internal field ID:
```javascript
$(document).ready(function(){
  var val = $('[name="f_43406340"]').val();
  var num = parseInt(val); // Convert to number for calculations
});
```
Find field IDs via browser DevTools (inspect the field element).

**Cookies/localStorage for Session State**
Persist user preferences or filter state across page loads:
```javascript
// Save
localStorage.setItem("lastFilter", filterValue);
// Restore
var saved = localStorage.getItem("lastFilter");
if (saved) { /* apply filter */ }
```

**AI Data Analysis (New Feature, Jan 2026)**
TeamDesk now includes built-in AI analysis per table. Configure custom instructions at Setup > Tables > [Table] > AI Instructions to guide the AI's analysis context (e.g., "This table tracks solar energy generation. kWh values should never be negative.").

### Document Generation

**CardText for Numbers in Words (Word Merge Documents)**
Display numeric amounts as words in generated Word/PDF documents:
```
{ ={ MERGEFIELD Amount } \* CardText }
```
Outputs: "One Thousand Five Hundred" for 1500. Only works in English. For Portuguese/Spanish, use the auxiliary table approach above.

**Dynamic Checkboxes in Merge Documents**
```
{ IF { MERGEFIELD Active } = "true" "☑ Active" "☐ Active" }
```

**Subcategories/Subtotals in Documents**
For grouped data in merge documents, use a many-to-many relationship as an intermediary:
1. Create a "Category" table with category names
2. Link detail records to categories via many-to-many
3. In the merge document, iterate categories first, then details within each category
4. Each category section gets its own subtotal

### Backup & Restore CLI (`td` tool)

Cross-platform console tool (Windows/macOS/Linux, Intel/AMD/ARM). Download from KB article #856.

**Basic commands:**
```bash
td backup <url> -u=<token> -f=<folder> -v          # Full backup
td restore <url> -u=<token> -f=<mapping-file> -v    # Restore from CSV
```

The `<url>` accepts multiple formats — all equivalent:
- `101885` (just the database ID)
- `https://www.teamdesk.net/secure/api/v2/101885/`
- `https://www.teamdesk.net/secure/db/101885/overview.aspx`

**Key options:**

| Option | Description |
|--------|-------------|
| `-u=<token>` | API token (or email). Restore requires Manage Data privilege |
| `-f=<folder>` | Backup folder. Supports date patterns: `-f=backup_{0:yyyy-MM-dd}` |
| `-t=<table>` | Backup/restore specific table(s). Repeat for multiple: `-t=Invoice -t=Item` |
| `-X=<table>` | Exclude table(s): `-X="Audit Log" -X="Large Table"` |
| `--culture=<locale>` | Locale for date/number parsing. **Use `pt-BR` for DD/MM/YYYY and comma decimals** |
| `-v` | Verbose progress output |
| `-b` | Stop on first error (default: recover and continue) |
| `--encoding=<enc>` | CSV encoding (default: UTF-8) |
| `-d=<char>` | Column delimiter (default: TAB) |

**Incremental behavior:** On subsequent runs, the tool compares the stored `.tdbackup` structure with the current schema. If unchanged, it only downloads new/modified records — significantly faster than full backup.

**Restore error handling:** Records that fail validation are written to a separate error file alongside the original CSV, with the error description as the last column. The original file is left intact for re-processing.

**Custom CSV mapping file (.tdbackup or .txt):**
Create custom imports from any CSV source using this mapping format:
```
; Comments start with semicolon
Invoices.csv -> Invoice
Invoice # -> Id
Address -> Address

Items.csv -> Item
* -> *
Ignore This Column ->
```

Mapping rules:
- `File.csv -> TableName` — maps CSV file to a table (use singular table name)
- `CSV Column -> TD Column` — maps a specific column
- `* -> *` — auto-match all columns by name
- `Column ->` — explicitly ignore a column (overrides `*->*`)
- Empty line separates table blocks

**Two-Factor Authentication**
Only available via SSO/SAML 2.0 in Enterprise Edition. Standard TeamDesk does not support 2FA natively. Workaround: use a password-protected page (external) that embeds TeamDesk via iframe.

### Import & Matching Patterns

**Multi-Column Matching on Import**
When importing data that may match existing records by different identifiers:
1. Create multiple many-to-many relationships (one per matching criterion)
2. Use Summary column `Count([Matched Records])` to identify matches
3. Filter for records where any match count > 0
4. Process matches in order of specificity (exact ID > email > name)

**Web-to-Record with Auto-Matching**
For forms that need to link to existing records without user selection:
1. Form captures identifying fields (email, phone, name)
2. Record Create trigger fires
3. Many-to-many relationship auto-links by matching criteria
4. Summary "Number of Matched Records" confirms the link

## Limits

- Select: 500 records default (use pagination)
- Export to Excel: 10,000 records
- Export to CSV: 100,000 records
- Trigger processing: 100 records per licensed user + 1,000 for external users pack
- SQL Server backend: Millions of records per table supported
