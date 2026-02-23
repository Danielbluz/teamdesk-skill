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

### Download
```
GET /api/v2/{db_id}/{table}/{column}/attachment?id={row_id}
GET /api/v2/{db_id}/{table}/{column}/attachment/{guid}
```
Returns binary file with appropriate Content-Type. HEAD also supported (metadata only).

### Upload (via multipart/related on Create/Update/Upsert)

There is **NO** dedicated upload endpoint. The `POST /{table}/{column}/attachment` endpoint does NOT exist (returns 405).

File uploads work **inline** with Create/Update/Upsert using `Content-Type: multipart/related`:

```
POST /api/v2/{db_id}/{table}/update.json
Authorization: Bearer {token}
Content-Type: multipart/related;boundary={boundary}

--{boundary}
Content-Type: application/json;charset=UTF-8

[{"@row.id": 42, "FileColumn": "cid:{content-id}"}]
--{boundary}
Content-Type: application/pdf
Content-Disposition: attachment;filename*=UTF-8''{url-encoded-filename}
Content-ID: {content-id}

{binary file data}
--{boundary}--
```

**Key details:**
- Use `@row.id` (NOT `Id`) to identify records in update.json
- `cid:{content-id}` in the JSON references the file MIME part by its Content-ID
- Works with create.json, update.json, and upsert.json
- Multiple files: add extra MIME parts with unique Content-IDs
- CSV import (`td import`) does NOT support attachment columns
- Ref: [ForeSoftCorp/TeamDesk-RESTAPI-PHP](https://github.com/ForeSoftCorp/TeamDesk-RESTAPI-PHP)

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

## Output Formats

- `.json` - JSON (recommended)
- `.xml` - XML
- `.html` - HTML table (useful for Excel live connection)
