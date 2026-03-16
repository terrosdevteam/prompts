## INTEGRATION PROXY

The integration proxy is an internal HelloLeo Webapp backend service that securely handles API calls to external services (like Supabase Self-Hosted, Odoo, Airtable, etc.) using stored credentials. It prevents CORS issues and keeps API keys secure.

**Base URL Variable:**

```javascript
const INTEGRATION_PROXY_URL = 'https://api.app.helloleo.dev/api/integration-proxy/${PROJECT_UUID}';
```

**Required Parameters:**

- `integration`: Name of the integration (e.g., 'supabase-self-hosted', 'odoo', 'airtable')
- `endpoint`: API endpoint path (e.g., '/products', '/users')
- `method`: HTTP method ('GET', 'POST', 'PATCH', 'DELETE')
- `body`: Request payload (varies by method)
- `headers`: Optional headers (including Authorization tokens)

### **Supabase Self-Hosted API Calls - Use Integration Proxy**

When calling Supabase Self-Hosted APIs, always use the integration proxy with this format:

```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'supabase-self-hosted',
    endpoint: '/products',  // Table name (without /rest/v1 prefix)
    method: 'GET',         // HTTP method
    headers: {             // Optional: include auth tokens for protected routes
      'Authorization': 'Bearer {user_token}'
    },
    body: {                // For GET: query parameters; For POST/PATCH: JSON data
      select: 'id,name,price',
      limit: 10
    }
  })
})
```

### **Key Rules:**

1. **Always use the INTEGRATION_PROXY_URL** - Never call Supabase directly
2. **Endpoint format**: Use table name only (e.g., `/products`), not `/rest/v1/products` - the proxy adds `/rest/v1/` automatically
3. **GET requests**: Put query parameters (`select`, `filter`, `order`, `limit`, `offset`) in the `body` object - they will be converted to URL query parameters
4. **POST/PATCH requests**: Put data directly in the `body` object as JSON
5. **DELETE requests**: Use filter syntax in `body` or endpoint URL parameters
6. **No automatic auth headers** - Routes are either public or require `Authorization` header from frontend

### **Authentication**

Supabase Self-Hosted routes are either **public** (no auth required) or **protected** (require user authentication token).

**For Protected Routes:**

Include the user's authentication token in the `headers` object:

```javascript
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'supabase-self-hosted',
    endpoint: '/products',
    method: 'GET',
    headers: {
      'Authorization': 'Bearer {user_token}' // User token from Supabase Auth
    },
    body: {
      select: '*'
    }
  })
})
```

**Important Notes:**

- **Public routes**: Omit `Authorization` header - request will work without authentication
- **Protected routes**: Must include `Authorization: Bearer {user_token}` header
- The proxy does NOT automatically add `apikey` or `anon_key` headers - authentication is handled entirely by the frontend
- User tokens come from Supabase Auth (frontend login) and should be passed through in the `headers` object

### **Query Parameter Format**

For GET requests, the proxy converts `body` parameters to Supabase PostgREST query parameters:

- `select`: Field selection (e.g., `'id,name,price'` or `'*'`)
- `filter`: Object with `column` and `value` (e.g., `{ column: 'price', value: 'gt.50' }`)
- `order`: Ordering (e.g., `'price.desc'` or `'created_at.asc'`)
- `limit`: Number of records (e.g., `10`)
- `offset`: Pagination offset (e.g., `0`)
- `range`: Array `[from, to]` - converted to `limit` and `offset`

**PostgREST Filter Operators:**

- `eq`: equals
- `gt`: greater than
- `gte`: greater than or equal
- `lt`: less than
- `lte`: less than or equal
- `neq`: not equal
- `like`: pattern matching
- `ilike`: case-insensitive pattern matching
- `in`: value in array

### **Response Format**

The integration proxy wraps ALL responses in this structure:

```javascript
{
  success: true,
  status: 200,
  data: [...] // Direct Supabase response (array for queries, object for single records)
}
```

**IMPORTANT:** Always unwrap the proxy response before accessing the actual data:

```javascript
const result = await fetch(INTEGRATION_PROXY_URL, { ... }).then(r => r.json());

if (!result.success) throw new Error(`API error: ${result.status}`);
const records = result.data || []; // Access actual Supabase data via result.data
```

## Edge Functions

Edge Functions are server-side TypeScript functions that run on Deno. They're useful for tasks requiring server-side logic, secrets, or database operations that can't be done securely in the browser.

**Two separate systems:**
1. **Function Management** - Deploy/list/delete functions available via MCP tools
2. **Function Invocation** - Execute functions via the Edge Runtime

---

### **Function Invocation (Edge Runtime)**

Functions are invoked through the Edge Runtime, **not** the Bridge API.

**Base URL Variable:**

```javascript
const EDGE_FUNCTIONS_URL = `${SUPABASE_URL}/functions/v1`;
```

**Invoke a Function:**

```javascript
fetch(`${EDGE_FUNCTIONS_URL}/{function-name}`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer {SUPABASE_ANON_KEY}'  // or user JWT
  },
  body: JSON.stringify({
    // Your payload
  })
})
```

---

### **Writing Edge Functions**

Edge Functions use Deno's `Deno.serve()` API. Use the `@supabase/supabase-js` library for database operations:

```typescript
import { createClient } from "jsr:@supabase/supabase-js@2";

Deno.serve(async (req) => {
  if (req.method !== 'POST') {
    return new Response(JSON.stringify({ error: 'Method not allowed' }), {
      status: 405,
      headers: { 'Content-Type': 'application/json' }
    })
  }

  const supabase = createClient(
    Deno.env.get("SUPABASE_URL")!,
    Deno.env.get("SUPABASE_SERVICE_ROLE_KEY")!
  );

  try {
    const body = await req.json();

    // INSERT
    const { data: inserted, error: insertError } = await supabase
      .from("products")
      .insert({ name: body.name, price: body.price })
      .select();

    // SELECT
    const { data: products, error: selectError } = await supabase
      .from("products")
      .select("*")
      .eq("category", "electronics")
      .order("price", { ascending: true })
      .limit(10);

    // UPDATE
    const { data: updated, error: updateError } = await supabase
      .from("products")
      .update({ price: 149 })
      .eq("id", body.id)
      .select();

    // DELETE
    const { error: deleteError } = await supabase
      .from("products")
      .delete()
      .eq("id", body.id);

    return new Response(JSON.stringify({ success: true, data: products }), {
      headers: { "Content-Type": "application/json" }
    });
  } catch (error) {
    return new Response(JSON.stringify({ error: error.message }), {
      status: 500,
      headers: { "Content-Type": "application/json" }
    });
  }
});
```

**Import syntax for Deno:**
```typescript
import { createClient } from "jsr:@supabase/supabase-js@2";
```

### **Available Environment Variables**

Inside edge functions, these environment variables are available via `Deno.env.get()`:

| Variable | Description |
|----------|-------------|
| `SUPABASE_URL` | Your Supabase API URL |
| `SUPABASE_ANON_KEY` | Public anon key (respects RLS) |
| `SUPABASE_SERVICE_ROLE_KEY` | Service role key (bypasses RLS) |

**When to use which key:**
- Use `SUPABASE_ANON_KEY` when you want RLS policies to apply
- Use `SUPABASE_SERVICE_ROLE_KEY` for admin operations that bypass RLS

### **Alternative: Raw Fetch Calls**

If you prefer not to use the client library, you can make raw fetch calls. **Important:** You must include both `apikey` and `Authorization` headers:

```typescript
const supabaseUrl = Deno.env.get('SUPABASE_URL')
const anonKey = Deno.env.get('SUPABASE_ANON_KEY')

const response = await fetch(`${supabaseUrl}/rest/v1/products?select=*`, {
  method: 'GET',
  headers: {
    'Content-Type': 'application/json',
    'apikey': anonKey,                    // Required!
    'Authorization': `Bearer ${anonKey}`, // Required!
  }
})
```

**Note:** Omitting the `apikey` header will result in `"No API key found in request"` errors.

### **Key Rules**

1. **Management vs Invocation** - Use Bridge API (`/functions/v1/manage/`) to deploy/manage, use Edge Runtime (`/functions/v1/`) to invoke
2. **Use the Supabase client library** - Cleaner code and handles auth headers automatically
3. **Raw fetch requires both headers** - If using fetch directly, include both `apikey` and `Authorization` headers
4. **Use service role key for admin operations** - For bypassing RLS or admin tasks
5. **Functions are Deno** - Use `Deno.serve()`, `Deno.env.get()`, and `jsr:` imports, not Node.js APIs
6. **Hot reload** - Functions are automatically picked up by the runtime after deployment
