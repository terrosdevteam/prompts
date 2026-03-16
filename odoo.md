# Odoo Integration Instructions

**CRITICAL:** When THE USER mentions Odoo, start with MCP exploration FIRST. Do NOT check filesystem. Do NOT use bash.

---

## MCP COMMANDS (Backend Exploration)

**Available Tool:** The Odoo MCP has ONE tool: `execute_method`

**Do NOT attempt:** `backend_details`, `odoo_connect`, or any other MCP commands.

**Parameters:**
- `model` (string, required): Odoo model name (e.g., 'res.partner', 'sale.order', 'product.product')
- `method` (string, required): Method name (e.g., 'search_read', 'create', 'write', 'unlink', 'read_group')
- `args` (list, optional): Positional arguments
- `kwargs` (dict, optional): Keyword arguments

**Common Methods:**
- **Read**: `search_read` with domain in `args`
- **Create**: `create` with data in `args[0]`
- **Update**: `write` with `[ids, data]` in `args`
- **Delete**: `unlink` with `[ids]` in `args`
- **Aggregate**: `read_group` for KPIs/charts
  - Aggregations: `:sum`, `:avg`, `:min`, `:max`, `:count`
  - Groupby: `field_name`, `field_name:day`, `field_name:month`, `field_name:year`
  - **Use `read_group` for:** KPIs, charts, totals (returns KB-sized data)
  - **Use `search_read` for:** tables, lists (with `limit: 100`)

**MCP Examples:**

```python
# Explore model structure
execute_method(model='ir.model.fields', method='search_read', args=[[['model', '=', 'sale.order']]], kwargs={'fields': ['name', 'ttype', 'required'], 'limit': 100})

# Search partners
execute_method(model='res.partner', method='search_read', args=[[['is_company', '=', True]]], kwargs={'fields': ['name', 'email'], 'limit': 10})

# Create product
execute_method(model='product.template', method='create', args=[{'name': 'New Product', 'list_price': 100.0}])

# Update product
execute_method(model='product.template', method='write', args=[[123], {'list_price': 150.0}])

# Aggregate sales
execute_method(model='sale.order', method='read_group', args=[[['state', '=', 'sale']]], kwargs={'fields': ['amount_total:sum'], 'groupby': ['date_order:month']})

# Batch query
execute_method(model='product.product', method='search_read', args=[[['id', 'in', [1, 2, 3, 4, 5]]]], kwargs={'fields': ['id', 'name', 'type']})
```

**MCP Rules:**
- ✅ **ALWAYS exclude `image*` fields** (image_1920, image_512, etc.) from `kwargs.fields` — breaks context limits. Frontend: OK to include for display (browser handles base64).
- ✅ **Bulk operations in ONE call** — Create multiple records with `create([{...}, {...}])`, never loop
- ✅ **Update LEO_RULES.md IMMEDIATELY** after discovering schema — this is MANDATORY
- ✅ **Always paginate** with `limit` and `offset`
- ⚠️ **Known invalid fields:** `date_planned_start` (not in `mrp.production` model)

---

## INTEGRATION PROXY (Frontend Runtime)

Frontend code calls Odoo via Integration Proxy (not MCP). Proxy handles auth/CORS.

**Base URL:**
```javascript
const INTEGRATION_PROXY_URL = 'https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}';
```

**Odoo-specific rules:**
1. Endpoint is ALWAYS `/jsonrpc`
2. Use `search_read`, NOT `web_search_read`
3. Domain goes in `args`, NOT `kwargs.domain`
4. Use `fields` array, NOT `specification` object
5. NEVER wrap in JSON-RPC format — proxy converts `{model, method, args, kwargs}` → JSON-RPC + auth automatically. Raw JSON-RPC causes auth failure.

**Response:** `result.data.result` (NOT `result.data` — account for proxy wrapper `{ success, data: { result } }`).

---

## ⚠️ N+1 QUERY ANTI-PATTERN

**NEVER query inside loops.**

**❌ FORBIDDEN:**
```javascript
for (const line of lines) {
  await fetch(PROXY, { body: JSON.stringify({ model: 'product.product', method: 'search_read', args: [[['id', '=', line.product_id[0]]]] }) })
}
```

**✅ Option 1: Relational Filters** (best for filtering)
```javascript
{ model: 'sale.order.line', method: 'search_read',
  args: [[['state', '=', 'sale'], ['product_id.type', 'in', ['product', 'consu']]]],  // Dot notation!
  kwargs: { fields: ['id', 'product_id'], limit: 100 } }
```

**✅ Option 2: Batch Queries** (best for fetching related records)
```javascript
const productIds = [...new Set(lines.map(line => line.product_id[0]))]
// Single query with 'in' operator
{ model: 'product.product', method: 'search_read', args: [[['id', 'in', productIds]]], kwargs: { fields: ['id', 'type'] } }
```

---

## AUTHENTICATION (Portal Users)

**Portal login:** Collect ONLY `username` and `password`. Database name auto-filled from project config.

**This is the ONLY case where JSON-RPC format is required** (proxy detects auth by `service: 'common'`):
```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'odoo', endpoint: '/jsonrpc', method: 'POST',
    body: {
      jsonrpc: '2.0', method: 'call',
      params: { service: 'common', method: 'authenticate', args: [null, username, password, {}] },  // null = proxy fills db
      id: 1
    }
  })
})
// Response: result.data.result → { uid, username, name, user_context }
```

---

## AGGREGATION & KPIS

Use `read_group` for dashboards:

```javascript
body: {
  model: 'account.move', method: 'read_group',
  args: [[['move_type', '=', 'out_invoice'], ['state', '=', 'posted']]],
  kwargs: { fields: ['amount_total:sum', 'amount_untaxed:sum'], groupby: ['invoice_date:month'], lazy: false }
}
```

---

## PAGINATION (MANDATORY)

**NEVER query without `limit`.** Always use `limit` and `offset` in `kwargs`. Large datasets cause timeouts and memory issues.
