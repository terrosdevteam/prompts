## Notion MCP

- When using the MCP call API-post-database-query, you must set the page_size param to 10 to avoid flooding your context window (unless you need to see everything and you know the database in not too big).

## Notion Integration Proxy Instructions

**Base URL Variable:**
```
const INTEGRATION_PROXY_URL = 'https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}';
```

**Common Notion Methods:**
- **Read**: GET pages, databases, or search
- **Create**: POST pages or database entries
- **Update**: PATCH pages or database entries
- **Delete**: DELETE pages or database entries

## Examples

### Example - Create Page:
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'notion',
    endpoint: '/v1/pages',
    method: 'POST',
    body: {
      parent: {
        database_id: "database_id_here"
      },
      properties: {
        "Name": {
          "title": [
            {
              "text": {
                "content": "New Page Title"
              }
            }
          ]
        },
        "Status": {
          "select": {
            "name": "Active"
          }
        }
      }
    }
  })
})
```

### Example - Update Page:
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'notion',
    endpoint: '/v1/pages/{page_id}',
    method: 'PATCH',
    body: {
      properties: {
        "Status": {
          "select": {
            "name": "Completed"
          }
        }
      }
    }
  })
})
```

### Example - Query Database:
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'notion',
    endpoint: '/v1/databases/{database_id}/query',
    method: 'POST',
    body: {
      filter: {
        "property": "Status",
        "select": {
          "equals": "Active"
        }
      },
      page_size: 10
    }
  })
})
```

### Example - Search:
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'notion',
    endpoint: '/v1/search',
    method: 'POST',
    body: {
      query: "search term",
      filter: {
        "value": "page",
        "property": "object"
      },
      page_size: 10
    }
  })
})
```

### Example - Get Page Content:
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'notion',
    endpoint: '/v1/pages/{page_id}',
    method: 'GET'
  })
})
```

### Example - Delete Page:
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'notion',
    endpoint: '/v1/pages/{page_id}',
    method: 'DELETE'
  })
})
```

## Key Rules:

1. **Always use** `INTEGRATION_PROXY_URL` - Never call Notion directly
2. **Use** `/v1/` endpoint format for all Notion API calls
3. **Replace** `{page_id}`, `{database_id}` with actual IDs
4. **Use** `properties` object for page/database entries
5. **Use** `filter` object for querying with conditions
6. **Use** `page_size` parameter to limit results (max 100)

## API Response Structure:
The integration proxy returns responses in this format:
```json
{
  "success": boolean,
  "status": number,
  "data": {
    "results": [...],
    "next_cursor": "string",
    "has_more": boolean
  }
}
```

Always access the actual Notion data via `response.data` first, then `response.data.results`:
```javascript
const result = await response.json();
const pages = result.data?.results || [];
```

## Miscellaneous:
- Notion has rate limits (3 requests/second), so batch operations when possible
- Page IDs are UUIDs (32 characters with hyphens)
- Database IDs are UUIDs (32 characters with hyphens)
- Property names are case-sensitive and must match exactly
- Notion API returns: `{results: [...], next_cursor?: string, has_more: boolean}` structure
- Always handle the `next_cursor` parameter for pagination of large datasets
- Use `has_more` to determine if there are more results to fetch
