If THE USER just activated the Airtable MCP, always provide the mcp call to check the availables bases first

When using the MCP call `list_tables` you must set the `describe_table` parameters. Here are the options:
- `tableIdentifiersOnly`: Returns only `id` and `name` of each table.
- `identifiersOnly`: Returns `id`, `name` of the table as well as `id` and `name` of all fields and views.
- `full`: Returns everything
For most USER, the number of Airtable table will be consequent and using `identifiersOnly` will flood your context window. You MUST always default to `detailLevel="tableIdentifiersOnly"`. You can then request individual table information or decide to use `identifiersOnly`.
Similarly, when using the MCP call `describe_table`, default to `detailLevel="identifiersOnly"`.

When using the MCP call `list_records` or `search_records`, you must set the `maxRecords` param to 3 to avoid flooding your context window.

## Airtable Integration Proxy Instructions

### Common Airtable Methods:
- **Read**: GET records with optional filtering
- **Create**: POST records with data
- **Update**: PATCH records with record IDs and data  
- **Delete**: DELETE records with record IDs

### Examples

**Example - Create Record:**
```javascript
fetch('https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'airtable',
    endpoint: '/v0/{baseId}/{tableIdOrName}',
    method: 'POST',
    body: {
      records: [
        {
          fields: {
            "Name": "New Product",
            "Price": 100.0,
            "Status": "Active"
          }
        }
      ]
    }
  })
})
```

**Example - Update Record:**
```javascript
fetch('https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'airtable',
    endpoint: '/v0/{baseId}/{tableIdOrName}',
    method: 'PATCH',
    body: {
      records: [
        {
          id: "recXXXXXXXXXXXXXX",
          fields: {
            "Price": 150.0
          }
        }
      ]
    }
  })
})
```

**Example - List Records:**
```javascript
fetch('https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'airtable',
    endpoint: '/v0/{baseId}/{tableIdOrName}',
    method: 'GET',
    body: {
      maxRecords: 10,
      fields: ["Name", "Price", "Status"],
      filterByFormula: "AND({Status} = 'Active')"
    }
  })
})
```

**Example - Delete Record:**
```javascript
fetch('https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'airtable',
    endpoint: '/v0/{baseId}/{tableIdOrName}',
    method: 'DELETE',
    body: {
      records: ["recXXXXXXXXXXXXXX", "recYYYYYYYYYYYYYY"]
    }
  })
})
```

### Key Rules:
- Always use `https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}` - Never call Airtable directly
- Use `/v0/{baseId}/{tableIdOrName}` endpoint format
- Replace `{baseId}` with actual base ID (starts with "app")
- Replace `{tableIdOrName}` with table name or ID
- Use `records` array for create/update operations
- Use `fields` object within each record
- For filtering, use `filterByFormula` with Airtable formula syntax
- For pagination, use `offset` parameter from previous response

### API Response Structure:

The integration proxy returns responses in this format: {success: boolean, status: number, data: {records: [...], offset?: string}}
Always access the actual Airtable data via response.data first, then response.data.records
Handle the nested structure properly: const result = await response.json(); const records = result.data?.records || []

### Miscellaneous:
- Airtable has rate limits (5 requests/second), so batch operations when possible
- Record IDs start with "rec" followed by 14 characters
- Base IDs start with "app" followed by 14 characters
- Field names are case-sensitive and must match exactly
- Airtable API returns: `{records: [...], offset?: string}` structure
- Always handle the `offset` parameter for pagination of large datasets
