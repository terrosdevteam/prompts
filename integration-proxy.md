# Integration Proxy (Frontend Runtime)

Frontend code calls integrations via the Integration Proxy (not MCP directly). The proxy handles authentication and CORS. NEVER use direct URLs to integration services or call MCP from frontend code.

**Base URL:** The `INTEGRATION_PROXY_URL` is provided in your integration-specific instructions (e.g., Odoo, Airtable). Use it as-is.

**Usage pattern (Odoo example):**
```jsx
const result = await fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'odoo', endpoint: '/jsonrpc', method: 'POST',
    body: { model: 'product.template', method: 'search_read',
      args: [[["type", "=", "consu"]]], kwargs: { fields: ['name', 'list_price'], limit: 10, offset: 0 } }
  })
}).then(r => r.json());
if (!result.success) throw new Error(result.error || `API error: ${result.status}`);
```

**Response unwrapping — ALL responses are wrapped in `{ success, status, data }`:**
- Odoo: `result.data.result` | Airtable: `result.data.records` | Notion: `result.data.results`
- Microsoft 365: `result.data.value` | Supabase/DimoMaint: `result.data` | Klaviyo: `result.data.data`

Common mistake: `data.result` instead of `data.data.result` (Odoo). Always account for the proxy wrapper.

**Rules:** Always paginate with `limit`/`offset`. Proxy-based integrations (Odoo, Airtable) use the proxy. SDK-based (Supabase, Xano) use their SDK/REST endpoints. Never call MCP from client code.
