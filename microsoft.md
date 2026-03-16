MCP COMMANDS
When to use: Use MCP commands to interact with Microsoft Dataverse (Power Platform / Dynamics 365).

**SYNTAX:** `mcp microsoft365 <command> {json_params}`

IMPORTANT RULES:
- Always use exactly this format: `mcp microsoft365 <command> {json_params}`
- NEVER use `mcp call`, `--params`, or any other format
- NEVER chain multiple MCP commands with && or ; - execute ONE command at a time
- Wait for each command result before running the next one

Available Commands:

**Data Operations:**
- list_environments - List available Dataverse environments
- list_tables - List tables in an environment (use custom_only:true to filter)
- get_table_schema - Get columns/schema of a table
- query_records - Query records with filters
- get_record - Get a single record by ID
- create_record - Create a new record
- update_record - Update an existing record
- delete_record - Delete a record
- fetch_xml_query - Execute FetchXML queries

**Schema Operations:**
- create_table - Create a new table (entity) in Dataverse
- add_column - Add a column (attribute) to an existing table
- delete_table - Delete a table (WARNING: deletes all data!)
- delete_column - Delete a column from a table

Important: The environment_url is automatically set when the user selects an environment in the UI. You don't need to pass it for most calls.

MCP Examples:

**List custom tables:**
mcp microsoft365 list_tables {"custom_only": true}

**Query records:**
mcp microsoft365 query_records {"table_name": "accounts", "select": "name,accountnumber", "top": 10}

**Create a record:**
mcp microsoft365 create_record {"table_name": "accounts", "data": {"name": "New Account"}}

**Create a new table:**
mcp microsoft365 create_table {"schema_name": "new_product", "display_name": "Product", "plural_name": "Products", "description": "Product catalog"}

**Add a column to a table:**
mcp microsoft365 add_column {"table_name": "new_product", "column_name": "new_price", "display_name": "Price", "column_type": "money"}

**Column types for add_column:** string, int, decimal, money, boolean, datetime, memo

Notes:
- Use custom_only: true with list_tables to see only user-created tables
- **Table name formats differ by command:**
  - `get_table_schema`: Use **logical_name** (singular) - e.g., `new_product`, `account`
  - `query_records`: Use **entity_set_name** (plural) - e.g., `new_products`, `accounts`
  - `create_record`, `update_record`, `delete_record`: Use **entity_set_name** (plural)
- For create_table, schema_name must include a publisher prefix (e.g., "new_", "cr123_")
- You can override environment_url if needed, but it's usually not necessary
- To modify a table/column, first use get_table_schema to see current structure, then delete and recreate (Dataverse requires PUT with full schema for updates)
- delete_table and delete_column permanently delete data - use with caution

---

# INTEGRATION PROXY

The integration proxy is an internal HelloLeo Webapp backend service that securely handles API calls to Dataverse using stored credentials. It prevents CORS issues and keeps tokens secure.

**Base URL Variable:**
```javascript
const INTEGRATION_PROXY_URL = 'https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}';
```

**Required Parameters:**
- `integration`: Always `'microsoft365'`
- `endpoint`: OData API path (e.g., `/accounts`, `/new_products`, `/contacts`)
- `method`: HTTP method (`'GET'`, `'POST'`, `'PATCH'`, `'DELETE'`)
- `body`: Request payload (OData query params for GET, record data for POST/PATCH)

## Dataverse API Calls - Use Integration Proxy

### **Read Records (GET):**
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'microsoft365',
    endpoint: '/accounts',  // Entity set name (plural)
    method: 'GET',
    body: {
      '$select': 'name,accountnumber,revenue',
      '$filter': "statecode eq 0",
      '$orderby': 'name asc',
      '$top': 50
    }
  })
})
```

### **Get Single Record:**
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'microsoft365',
    endpoint: '/accounts(00000000-0000-0000-0000-000000000001)',  // GUID in parentheses
    method: 'GET',
    body: {
      '$select': 'name,accountnumber'
    }
  })
})
```

### **Create Record (POST):**
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'microsoft365',
    endpoint: '/accounts',
    method: 'POST',
    body: {
      name: 'New Account',
      accountnumber: 'ACC-001',
      revenue: 50000
    }
  })
})
```

### **Update Record (PATCH):**
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'microsoft365',
    endpoint: '/accounts(00000000-0000-0000-0000-000000000001)',
    method: 'PATCH',
    body: {
      revenue: 75000
    }
  })
})
```

### **Delete Record:**
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'microsoft365',
    endpoint: '/accounts(00000000-0000-0000-0000-000000000001)',
    method: 'DELETE'
  })
})
```

## OData Query Parameters

Use these in the `body` object for GET requests:

| Parameter | Description | Example |
|-----------|-------------|---------|
| `$select` | Columns to return | `'name,accountnumber'` |
| `$filter` | OData filter expression | `"statecode eq 0"` |
| `$orderby` | Sort order | `'createdon desc'` |
| `$top` | Max records to return | `50` |
| `$skip` | Records to skip (pagination) | `100` |
| `$expand` | Expand related entities | `'primarycontactid'` |
| `$count` | Include total count | `true` |

## OData Filter Examples

```javascript
// Equals
"$filter": "statecode eq 0"

// Contains (string)
"$filter": "contains(name, 'Contoso')"

// Date comparison
"$filter": "createdon ge 2024-01-01"

// Multiple conditions
"$filter": "statecode eq 0 and revenue gt 10000"

// Lookup field
"$filter": "_primarycontactid_value eq 00000000-0000-0000-0000-000000000001"
```

## Response Format

```javascript
{
  success: true,
  status: 200,
  data: {
    "@odata.context": "...",
    "value": [
      { "name": "Account 1", "accountnumber": "ACC-001", ... },
      { "name": "Account 2", "accountnumber": "ACC-002", ... }
    ]
  }
}
```

**Access records:** `response.data.value`

## Key Rules

1. **Always use INTEGRATION_PROXY_URL** - Never call Dataverse directly
2. **Use entity set names** (plural) in endpoint: `/accounts`, `/contacts`, `/new_products`
3. **GUID format** for single records: `/accounts(00000000-0000-0000-0000-000000000001)`
4. **Always use `$top`** for pagination - never fetch unlimited records
5. **Lookup fields** use `_fieldname_value` format in filters

## Pagination Example

```javascript
async function fetchAllRecords(entitySet, filter, select) {
  const pageSize = 100;
  let skip = 0;
  let allRecords = [];

  while (true) {
    const response = await fetch(INTEGRATION_PROXY_URL, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        integration: 'microsoft365',
        endpoint: `/${entitySet}`,
        method: 'GET',
        body: {
          '$select': select,
          '$filter': filter,
          '$top': pageSize,
          '$skip': skip
        }
      })
    });
    const data = await response.json();
    const records = data.data.value || [];
    allRecords.push(...records);
    if (records.length < pageSize) break;
    skip += pageSize;
  }
  return allRecords;
}
```

---

# END-USER AUTHENTICATION (MSAL.js)

When users need their own Dataverse permissions (RBAC), use MSAL.js for end-user login.

**Package:** `@azure/msal-browser` and `@azure/msal-react`

**Config:**
- Client ID: `7c2f0083-1a78-4728-810e-21e84fed9812`
- Authority: `https://login.microsoftonline.com/common`
- Redirect URI: `window.location.origin`

**Get environment URL:** Run `mcp microsoft365 list_environments` to get the selected environment URL, then use it for the scope.

**MSAL Setup:**
```javascript
// main.tsx
import { PublicClientApplication } from '@azure/msal-browser'
import { MsalProvider } from '@azure/msal-react'

const msalConfig = {
  auth: {
    clientId: '7c2f0083-1a78-4728-810e-21e84fed9812',
    authority: 'https://login.microsoftonline.com/common',
    redirectUri: window.location.origin,
  },
}
const msalInstance = new PublicClientApplication(msalConfig)

// Wrap App with <MsalProvider instance={msalInstance}>
```

**Auth Service:**
```javascript
// Use the environment URL from list_environments for the scope
const DATAVERSE_SCOPE = 'https://YOUR_ENV.crm.dynamics.com/.default'  // Replace with actual env URL

export async function loginUser(msalInstance) {
  return msalInstance.loginPopup({ scopes: [DATAVERSE_SCOPE] })
}

export async function getAccessToken(msalInstance) {
  const accounts = msalInstance.getAllAccounts()
  if (accounts.length === 0) return null
  const response = await msalInstance.acquireTokenSilent({
    scopes: [DATAVERSE_SCOPE],
    account: accounts[0],
  })
  return response.accessToken
}
```

**Pass token to proxy:**
```javascript
const token = await getAccessToken(msalInstance)
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'microsoft365',
    endpoint: '/accounts',
    method: 'GET',
    userAccessToken: token,  // Enables per-user RBAC
    body: { '$select': 'name', '$top': 50 }
  })
})
```

If `userAccessToken` is provided, Dataverse applies that user's permissions. If omitted, uses project owner's token.

---

# DEBUGGING

**When Dataverse API calls fail, ask the user:**

> "Can you open the Network Monitor (bottom right of your screen), click on the Dataverse API call, and click 'Share with Leo'?"

**Common errors:**
- `401 Unauthorized` - Token expired, ask user to reconnect Microsoft 365 in project settings
- `404 Not Found` - Wrong entity set name (check plural form) or record doesn't exist
- `403 Forbidden` - User lacks permissions for this table/record
- `400 Bad Request` - Invalid OData query syntax

**NEVER ask the user to open DevTools or browser console.**
