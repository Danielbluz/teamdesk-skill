# TeamDesk REST API Reference

## Authentication

### Authorization Header (Recommended)
```
Authorization: Bearer {32-character-token}
```

All examples in this reference use the `Authorization` header. The token is a 32-character string generated at:
Setup > Database > Integration API > REST API Authorization Tokens > New

> **Security note:** TeamDesk also supports embedding the token in the URL path (`/api/v2/{db_id}/{token}/{table}/...`), but this is discouraged as URLs may appear in server logs, proxy logs, and browser history. Always prefer the `Authorization: Bearer` header.

## Select Method

Query records from table or view.

### From Table
```
GET /api/v2/{db_id}/{table}/select.json?parameters
```

### From View
```
GET /api/v2/{db_id}/{table}/{view}/select.json?parameters
```

### Parameters

| Parameter | Description | Example |
|-----------|-------------|---------|
| column | Columns to return (repeat for multiple, * for all updateable) | `column=Name&column=Email` |
| filter | Formula expression for filtering | `filter=[Status]="Active"` |
| sort | Column with optional //ASC or //DESC | `sort=Date//DESC` |
| top | Max records to return (default 500) | `top=100` |
| skip | Records to skip for pagination | `skip=500` |

### Response Format (JSON)
```json
[
  {
    "@row.id": 12,
    "@row.allow": "Edit, Delete",
    "Id": "60",
    "Name": "Example",
    "Date": "2024-01-15T00:00:00+00:00",
    "Checkbox": true,
    "Number": 1234.56
  }
]
```

### Data Type Mappings
- **Autonumber/Text**: String
- **Number/Currency/Percent**: Number
- **Checkbox**: Boolean (true/false)
- **Date**: "YYYY-MM-DDT00:00:00+00:00" (UTC midnight)
- **Time**: "0001-01-01THH:MM:SS+00:00"
- **Timestamp**: "YYYY-MM-DDTHH:MM:SS+ZZ:ZZ" (user timezone)
- **Duration**: Number (seconds)
- **User**: "Name <email>"
- **Null**: null

## Retrieve Method

Get specific record(s) by ID or key column.

```
GET /api/v2/{db_id}/{table}/retrieve.json?id={row_id}
GET /api/v2/{db_id}/{table}/retrieve.json?{KeyColumn}={value}
```

Multiple records:
```
GET /retrieve.json?id=1&id=2&id=3
```

## Create Method

Create new records.

```
POST /api/v2/{db_id}/{table}/create.json
Content-Type: application/json

[
  {"Name": "Record 1", "Value": 100},
  {"Name": "Record 2", "Value": 200}
]
```

### Response
```json
[
  {"@row.id": 123, "@row.status": "Created"},
  {"@row.id": 124, "@row.status": "Created"}
]
```

### Options
- `?triggers=false` - Disable workflow triggers (requires Manage Data privilege)

## Update Method

Update existing records.

```
POST /api/v2/{db_id}/{table}/update.json
Content-Type: application/json

[
  {"@row.id": 123, "Status": "Complete"},
  {"Id": "ABC", "Status": "Complete"}  // Using key column
]
```

## Upsert Method

Create or update based on match column.

```
POST /api/v2/{db_id}/{table}/upsert.json?match={ColumnName}
Content-Type: application/json

[
  {"ExternalId": "EXT001", "Name": "Updated Name", "Value": 150}
]
```

- If record with matching ExternalId exists: Update
- If no match: Create new record

## Delete Method

```
GET /api/v2/{db_id}/{table}/delete.json?id={row_id}
POST /api/v2/{db_id}/{table}/delete.json
Content-Type: application/json
[{"@row.id": 123}, {"@row.id": 124}]
```

## Describe Method

Get table/column metadata.

```
GET /api/v2/{db_id}/{table}/describe.json
```

### Response
```json
{
  "Name": "Contacts",
  "Columns": [
    {"Name": "Id", "Type": "Autonumber", "Key": true},
    {"Name": "Name", "Type": "Text", "Required": true},
    {"Name": "Email", "Type": "E-mail Address"}
  ]
}
```

## Attachment Method

Download/upload file attachments.

### Download
```
GET /api/v2/{db_id}/{table}/{column}/attachment?id={row_id}
GET /api/v2/{db_id}/{table}/{column}/attachment/{guid}
```

### Upload
```
POST /api/v2/{db_id}/{table}/{column}/attachment?id={row_id}
Content-Type: multipart/form-data
```

## User Method

Get current API user info.

```
GET /api/v2/{db_id}/-/user.json
```

## Error Handling

### HTTP Status Codes
- 200: Success
- 400: Bad request (invalid parameters)
- 401: Unauthorized (invalid token)
- 403: Forbidden (insufficient permissions)
- 404: Not found (table/view/record doesn't exist)
- 500: Server error

### Error Response
```json
{
  "Error": "Error message description"
}
```

### Common Errors
- "Column not found": Check column name spelling and case
- "Access denied": Token user lacks permission
- "Invalid filter": Formula syntax error in filter parameter

## Pagination Pattern

```javascript
async function getAllRecords(baseUrl) {
  let all = [];
  let skip = 0;
  const top = 500;
  
  while (true) {
    const url = `${baseUrl}?top=${top}&skip=${skip}`;
    const response = await fetch(url);
    const data = await response.json();
    
    if (data.length === 0) break;
    
    all = all.concat(data);
    skip += top;
    
    if (data.length < top) break;
  }
  
  return all;
}
```

## Caching

TeamDesk supports conditional caching:
- Response includes ETag header
- Send `If-None-Match: {etag}` to check for updates
- Returns 304 Not Modified if unchanged

## CORS Support

TeamDesk API supports Cross-Origin Resource Sharing for browser-based requests.

## Hidden/Undocumented Endpoints

### Export View as Excel/CSV
Download a view's data as a file (useful for scheduled reports via n8n or Call URL):
```
GET {URLRoot()}/exportview.aspx/report.xlsx?format=2&id={ViewID}
```

| Format | Value | Extension |
|--------|-------|-----------|
| CSV | 0 | .csv |
| XLSX | 2 | .xlsx |

The `ViewID` is found in the URL when viewing it in TeamDesk (after `id=`). The filename in the URL path (e.g., `report.xlsx`) becomes the downloaded file's name.

### Gateway Auto-Login (Chaining)
Authenticate and redirect in one request (for automated downloads or iframe embedding):
```
GET https://www.teamdesk.net/secure/gateway.aspx?action=Login&email={email}&password={password}&ReturnURL={encoded_url}
```

> **Security Warning:** Password is sent in plaintext in the URL. Only use in server-side automation (n8n Call URL), never in client-side code or shared links.

### Email to Database
Native feature (no API needed): Setup > Tools > Email to Database creates a unique email address per table. Incoming emails are automatically parsed into records:
- Subject → Text column
- Body → Memo column
- From → Email column
- Attachments → File Attachment column

Configure column mappings in the setup wizard. Useful for receiving automated reports (e.g., FusionSolar daily emails).

## Filter Variables in Call URL

When using dynamic values in API filter parameters, always URLEncode the value:
```
GET /api/v2/{db_id}/{table}/select.json?filter=Contains([Name], '` & URLEncode([SearchTerm]) & `')
```

For date filters:
```
filter=[Date]>=ToDate("` & Format([StartDate], "yyyy-MM-dd") & `")
```

## Output Formats

- `.json` - JSON (recommended)
- `.xml` - XML
- `.html` - HTML table (useful for Excel live connection)
