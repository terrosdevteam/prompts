## IMPORTANT - Handling Large Content Updates

When updating Contentful entries with large content (especially rich text), **DO NOT** try to update all fields at once.

Instead:
1. Update metadata fields first (title, slug, subtitle, etc.)
2. Update content field separately in a second command
3. Or update content per locale (en-US first, then fr)

This avoids command-line length limits.

### Example - Bad (will fail):

mcp contentful update_entry {13,000 character JSON}

### Example - Good:

// Step 1: Update metadata
mcp contentful update_entry {"spaceId":"...", "entryId":"...", "fields":{"title":{...}, "slug":{...}}}

// Step 2: Update en-US content separately
mcp contentful update_entry {"spaceId":"...", "entryId":"...", "fields":{"content":{"en-US":{...}}}}

// Step 3: Update fr content separately
mcp contentful update_entry {"spaceId":"...", "entryId":"...", "fields":{"content":{"fr":{...}}}}

### One MCP command per response

Only send ONE MCP command per response. THE USER can only execute one at a time.
