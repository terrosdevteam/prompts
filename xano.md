# Xano MCP

## 1. General Principles

**⚠️ MCP Response Display Bug:** When MCP response shows `data: "candidates"`, the API is correct (`data: $candidates`). This is a display bug - NEVER try to "fix" it. Accept as success and move on.

- Always use **MCP commands** to configure and interact with Xano.
- **Create items ONE AT A TIME** - never chain multiple `create_api`, `create_table`, or other creation commands with `&&` or in parallel. Wait for each command to complete before creating the next item.
- The `input` block is **required** for every script (`query`, `function`, etc.).
- To store data, use `db.add` — **never use** `db.insert`.
- **🚨 CRITICAL - `set_table_schema_fields` REPLACES entire schema:** `set_table_schema_fields` REPLACES the entire table schema, it does NOT add fields. To add/modify a field:
  1. **ALWAYS** call `get_table_schema` first to retrieve ALL existing fields
  2. Add your new field (or modify existing) to the complete field list
  3. **CRITICAL - NEVER include `id` field:** When calling `set_table_schema_fields`, NEVER include the `id` field in your fields list. The `id` field is managed automatically by Xano and should NEVER be modified. Including it can corrupt the table and break primary keys.
  4. Call `set_table_schema_fields` with the COMPLETE schema including ALL existing fields EXCEPT `id` (and `created_at` which is also auto-managed)
  **NEVER** call `set_table_schema_fields` with only new fields - this will DELETE all other fields and indexes, causing data loss and breaking primary keys.
- When using `update_api`, always use the **new API ID returned in the response**, as the system creates a new version rather than modifying the existing one.
- When setting up a Xano integration, extract the correct base URL from `backend_details` output (look for "base url: https://...") and use that exact URL as the default API in the frontend.

---

# XanoScript Basics

XanoScript is Xano's native scripting language for defining tables, APIs, functions, and backend logic.

Our XanoScript integration is still in beta, so if you have a lot of errors inform THE USER and suggest him to use the Xano Assistant interface directly in Xano.  

**Goal:** Generate valid XanoScript that can be pasted directly into Xano without syntax errors.

**Key capabilities:**
- Define tables, fields, indexes, and relationships
- Create or update APIs and functions with `query`, `function`, `stack`, `response` blocks
- Interact with external APIs or AI services via `fetch`
- Generate and deploy complete Xano backends programmatically


---

## Primitives

Top-level blocks (e.g. `query`, `function`, `task`, `table`) declare objects in Xano.

```
<primitive_keyword> <name> <attr>=<value> {
  ...
}

```

Examples:

```
query user_list verb=GET {
  auth = "user"
  ...
}

```

- Typical attributes: `verb=GET|POST|PUT|DELETE`, `auth="user"|false`
- Path params go in the name: `query user/{user_id} verb=GET { ... }`

---

## Input Block

Always include `input {}` even when no inputs exist.

```
input {
  text name
  int id?
  text[] tags
  email email filters=trim|lower
}

```

- `$input.<field>` references input values
- `?` marks optional fields
- `[]` marks arrays
- You can use filters like `trim|lower|min|max|capitalize`

---

## Stack & Functions

Use the `stack` block to execute logic.

Functions follow a `namespace.function` syntax and can return to variables with `as`.

```
stack {
  // Query all records
  db.query user {
    return = { type: "list" }
  } as $users

  // Query with filtering using search parameter
  db.query comment {
    search = $db.comment.user_id == $input.user_id
    return = { type: "list" }
  } as $user_comments

  // Get single record by specific field
  db.get user {
    field_name  = "id"
    field_value = $input.user_id
  } as $user
}

```

**Important database operation rules:**
- `db.query` → returns list, always specify `return = { type: "list" }`
- `db.query` supports `search` parameter for filtering: `search = $db.table.field == $value`
- `db.get` → returns single record, requires `field_name` and `field_value`
- Use `db.get` for lookups by specific field (e.g., by ID, email, etc.)
- Use `search` in `db.query` to filter results at query time
- Assign results with `as $variable`

---

## Response

Use **assignment syntax** — not a nested block.

```
response = $users

```

✅ Correct

```
response = $result

```

❌ Wrong

```
response { value = $result }

```

---

## Variables

```
var $formatted {
  value = []
}

var.update $formatted {
  value = $formatted|push:{"id":1}
}

// Ternary expressions
var $count {
  value = ($input.items != null) ? ($input.items.length) : 0
}

```

- Use `var` to create and `var.update` to modify
- Use `$variable` everywhere else
- Comments: `// single line`

---

## Dot Notation

Navigate objects, arrays, and database schemas:

```
$user.name
$users.0.email
$db.comment.user_id    // Access table schema for comparisons

```

---

## Filters

Filters manipulate data inline using pipes (`|`). Chainable and side-effect free.

```
"hello"|capitalize
"hello"|concat:" world"
$user.created_at|format_timestamp:"Y-m-d H:i:s"
$item|set:"name":($item.name|to_upper)    // set filter for object manipulation

```

---

## Array Operations

```
array.push $formatted_users {
  value = $formatted_item
}

```

---

## Loops

Use explicit `each as` blocks inside loops.

```
// For Each
foreach ($users) {
  each as $user {
    ...
  }
}

// For loop (0..N-1)
for (10) {
  each as $index {
    ...
  }
}

// While loop
while ($keep_running) {
  each {
    ...
  }
}

```

---

## Conditionals & Preconditions

```
conditional {
  if ($users|is_empty) {
    var.update $message { value = "No users" }
  }
  elseif ($users.length > 100) {
    ...
  }
  else {
    ...
  }
}

precondition ($model != null) {
  error_type = "notfound"
  error = "Not Found"
}

```

> precondition() raises error when condition is FALSE. Example: `precondition ($model != null)` throws if model IS null.
> 

---

## Settings (after response)

You can define primitive settings after `response = ...`:

```
response = $payload
history = 100

```

---

## Syntax Rules & Common Mistakes

**Critical Syntax Rules:**
- **NO COMMAS in object blocks** - XanoScript uses newlines, not commas to separate fields
  ```
  ✅ Correct:
  data = {
    name: $input.name
    email: $input.email
  }
  
  ❌ Wrong (causes variable corruption):
  data = {
    name: $input.name,
    email: $input.email,
  }
  ```
- **No semicolons**
- **Single quotes** preferred in strings
- **Prefix variables with `$`**
- **Do not use** JS features like arrow functions, destructuring, spread syntax
- `db.modify` and `db.update` **do not exist** — use `db.edit` or `db.patch`
- `db.delete` and `db.edit` require explicit `id = $input.id`
- When defining optional inputs, use `?` properly and defaults when possible
- **PATCH** is not a native verb — use `POST` or `PUT`
- **Prefer omitting `auth`** rather than setting `auth = false` to avoid syntax errors

**Common mistakes to avoid:**
- **Trying to "fix" APIs based on MCP response showing string literals** - if response shows `field: "field"`, the API is correct; don't update it
- **Using commas in data blocks** - this corrupts variable references (see example above)
- **Setting `auth = false`** - prefer omitting auth entirely to avoid syntax errors
- **Chaining multiple MCP creation commands** with `&&` - create items one at a time, waiting for each to complete
- **Placing `auth` after `response`** - by convention, place auth early (after opening brace, before or near input block)
- Missing `input {}` block or referencing inputs without `$input.`
- Using `response { value = ... }` instead of `response = ...`
- Omitting `return = { type: "list" }` on `db.query` when a list is expected
- Forgetting `as $var` after a function whose output you need later
- Using legacy parameter names (prefer `table` for `security.create_auth_token`)
- **Misusing `precondition` logic** - precondition throws error when condition is FALSE: `precondition ($model != null)` throws if model IS null

---

## 2. Table Declaration Example

```
table painting {
  auth = false

  schema {
    int id
    timestamp created_at? = now
    text example_text? filters=trim
    bool example_bool?
    password example_password?
    decimal example_decimal?
    email example_email? filters=trim|lower
    date example_date?
    uuid example_uuid?
    json example_json?
    attachment example_blob?
  }

  index = [
    { type: "primary", field: [{ name: "id" }] },
    { type: "gin", field: [{ name: "xdo", op: "jsonb_path_op" }] },
    { type: "btree", field: [{ name: "created_at", op: "desc" }] }
  ]
}

```

---

## 3. Query Examples

## 3.1 Dashboard Stats

```
query get_dashboard_stats verb=GET {
  description = 'Récupérer les statistiques du dashboard'

  input {}

  stack {
    db.query reservation { return = { type: "list" } } as $reservations
    db.query user { return = { type: "list" } } as $users
    db.query products { return = { type: "list" } } as $products
    db.query review { return = { type: "list" } } as $reviews
  }

  response = {
    success: true
    dashboard_data: {
      overview: {
        total_products: $products.length
        total_users: $users.length
        total_reservations: $reservations.length
        total_reviews: $reviews.length
      }
    }
  }
}

```

## 3.2 Product Search with Availability

```
query search_with_availability verb=GET {
  description = 'Recherche de produits avec vérification de disponibilité'

  input {
    text query
    text category
    date start_date
    date end_date
    decimal max_price
  }

  stack {
    db.query products { return = { type: "list" } } as $products
    db.query reservation { return = { type: "list" } } as $reservations
    db.query review { return = { type: "list" } } as $reviews
  }

  response = {
    success: true
    filters: {
      query: $input.query
      category: $input.category
      date_range: { start: $input.start_date, end: $input.end_date }
      max_price: $input.max_price
    }
    results: {
      products: $products
      reservations_context: $reservations
      reviews_context: $reviews
    }
  }
}

```

---

## 4. Data Creation Examples (`db.add`)

## 4.1 Create Reservation

```
query create_reservation verb=POST {
  description = "Créer une réservation"

  input {
    int user_id
    int product_id
    date start_date
    date end_date
  }

  stack {
    db.add reservation {
      data = {
        user: $input.user_id
        product: $input.product_id
        start_date: $input.start_date
        end_date: $input.end_date
        status: "pending"
      }
    } as $reservation
  }

  response = { success: true, data: $reservation }
}

```

## 4.2 Submit Feedback

```
query feedback verb=POST {
  description = "Add a feedback record"

  input {
    text name
    text email
    int rating
    text comment
  }

  stack {
    db.add feedback {
      data = {
        name: $input.name
        email: $input.email
        rating: $input.rating
        comment: $input.comment
      }
    } as $new_feedback
  }

  response = $new_feedback
}

```

---

## 5. APIs — Structure & Examples

APIs use the same `input`, `stack`, `response` structure covered in XanoScript Basics, plus HTTP-specific settings:

```
query <path_or_name> verb=GET|POST|PUT|DELETE {
  description = "What this API does"  // optional documentation
  auth = "user"                       // optional - convention is to place early
  
  input { ... }          // see Input Block above
  stack { ... }          // see Stack & Functions above  
  response = ...         // see Response above
  
  // Optional settings after response:
  tags = ["..."]
  history = 100
  cache = { ttl: 60, input: true, auth: true, datasource: false, ip: false }
}
```

**API-specific rules:**
- **`auth` handling:** Prefer omitting `auth` entirely unless you need to set it to `"user"` or another role. Setting `auth = false` can cause syntax errors.
- **If `auth` is specified**, convention is to place it early (immediately after opening brace, before or near `input`) for readability
- Path parameters live in the name: `query user/{user_id} verb=GET { ... }`
- Recommended order: **[auth] → input → stack → response → other settings (tags, history, cache)**
- Cache options: `ttl` (seconds), `input`, `auth`, `datasource`, `ip`, `headers: ["..."]`, `env: ["..."]`
- Caching tip: include only keys that materially change output (e.g., input and auth for per-user results)

### 5.1 Canonical API Templates

**A. Signup (POST) - No Auth Required**

```
// Signup and auth token
query auth/signup verb=POST {
  input {
    text name?
    email email? filters=trim|lower
    text password?
  }

  stack {
    db.get user {
      field_name  = "email"
      field_value = $input.email
    } as $existing

    precondition ($existing == null) {
      error_type = "accessdenied"
      error      = "This account is already in use."
    }

    db.add user {
      data = {
        created_at: "now"
        name      : $input.name
        email     : $input.email
        password  : $input.password
      }
    } as $user

    security.create_auth_token {
      table      = "user"
      id         = $user.id
      extras     = {}
      expiration = 86400
    } as $authToken
  }

  response = { authToken: $authToken }
  tags = ["auth"]
}

```

**B. Get by ID (GET with path param)**

```
// Get user by id
query user/{user_id} verb=GET {
  auth = "user"

  input {
    int user_id? filters=min:1
  }

  stack {
    db.get user {
      field_name  = "id"
      field_value = $input.user_id
    } as $model

    precondition ($model != null) {
      error_type = "notfound"
      error      = "Not Found"
    }
  }

  response = $model
  tags = ["users"]
}

```

**C. List (GET)**

```
// List users
query users verb=GET {
  input {}

  stack {
    db.query user {
      return = { type: "list" }
    } as $users
  }

  response = $users
  cache = { ttl: 30, input: false, auth: false }
}

```

**D. Get by Specific Field (GET with path param)**

```
// Get record by non-primary field (e.g., receipt_id)
query receipt_tax/{receipt_id} verb=GET {
  input {
    int receipt_id
  }

  stack {
    db.get receipt_tax {
      field_name  = "receipt_id"
      field_value = $input.receipt_id
    } as $tax
  
    precondition ($tax != null) {
      error_type = "notfound"
      error      = "No tax data found for this receipt"
    }
  }

  response = $tax
  tags = ["receipt", "tax"]
}

```

### 5.2 API Integration Example (OpenAI)

```
query ai_chat verb=POST {
  description = "AI chat for bicycle parts"

  input {
    text message
  }

  stack {
    var $ai_response {
      value = fetch("https://api.openai.com/v1/chat/completions", {
        method: "POST",
        headers: {
          "Authorization": "Bearer " + env.OPENAI_API_KEY,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          model: "gpt-3.5-turbo",
          messages: [
            { role: "system", content: "You are a bicycle parts expert." },
            { role: "user", content: $input.message }
          ],
          max_tokens: 300,
          temperature: 0.7
        })
      })
    }
  }

  response = {
    success: true
    message: $ai_response.choices.0.message.content
  }
}

```

---

## 6. Custom Functions — Structure & Examples

Functions use the same `input`, `stack`, `response` structure as APIs, but without HTTP-specific fields:

```
function <path/identifier> {
  description = "What this function does"  // optional documentation
  
  input { ... }          // see Input Block above
  stack { ... }          // see Stack & Functions above
  response = ...         // optional (omit for procedure-like functions)
  
  // optional settings:
  tags = ["..."]
  history = 100
}
```

**Function-specific rules:**
- `<path/identifier>` can be nested (e.g., `utilities/create_camel_case_slug`)
- Keep the order: **input → stack → response → settings**
- No HTTP `verb` or `auth` (functions are internal, not endpoints)
- Response is optional for procedure-like functions

### 6.1 Canonical Templates

#### 6.1.1 Function with a Return Value

```
// Return the number of items in a list
function utils/count_items {
  input {
    text[] items?
  }

  stack {
    var $count {
      value = ($input.items != null) ? ($input.items.length) : 0
    }
  }

  response = { count: $count }
  tags = ["utils"]
}

```

---

#### 6.1.2 Procedure-like Function (no Response)

```
// Log an event to the database (no direct return)
function audit/log_event {
  input {
    text action
    json details?
  }

  stack {
    db.add audit_log {
      data = {
        created_at: "now"
        action    : $input.action
        details   : $input.details
      }
    } as $log
  }

  tags = ["audit"]
}

```

---

#### 6.1.3 String → camelCase (Clear, Stepwise)

```
function utilities/to_camel_case {
  description = "Convert text to camelCase: strip non-alphanumerics, split, lowercase, then capitalize subsequent words."
  input {
    text text filters=trim
  }

  stack {
    // Normalize: remove non-alphanumerics (keep spaces), then lowercase
    var $normalized {
      value = $input.text
              |regex_replace:"/[^a-zA-Z0-9\s]/":""
              |to_lower
    }

    // Split on spaces
    var $words { value = $normalized|split:" " }

    // Initialize slug
    var $slug { value = "" }

    // Build camelCase (skip empties defensively)
    foreach ($words) {
      each as $w {
        conditional {
          if ($w != "") {
            conditional {
              // First token: append as-is
              if ($slug == "") {
                text.append $slug { value = $w }
              }
              else {
                // Subsequent tokens: capitalize first letter
                text.append $slug { value = $w|capitalize }
              }
            }
          }
        }
      }
    }
  }

  response = $slug
  tags = ["utils","string"]
}

```

Notes:

- The regex pattern uses `\s` for whitespace. Keep it quoted.
- Avoid inline lambda expressions; use `foreach` + `conditional`.

---

## 7. Authentication Flow (Xano)

- Login returns `authToken` (not `auth_token`)
- No user object in response — must call `/auth/me` to retrieve user
- Flow:
    1. Login → get `authToken`
    2. Store in `localStorage`
    3. Call `/auth/me`
    4. Store user object
    5. Use `user.id` in all authenticated requests
