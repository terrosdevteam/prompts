## **INTEGRATION PROXY - DimoMaint**

The integration proxy handles API calls to DimoMaint GMAO system using stored credentials (API key and tenant ID).

**Base URL Variable:**
```javascript
const INTEGRATION_PROXY_URL = 'https://api.preprod.app.helloleo.terros.io/api/integration-proxy/${PROJECT_UUID}';
```

**Required Parameters:**
- `integration`: Must be `'dimomaint'`
- `endpoint`: DimoMaint endpoint path (always starts with `/connector/`)
- `method`: HTTP method (`'GET'`, `'POST'`, `'PATCH'`, `'PUT'`)
- `body`: Request payload (for POST/PATCH/PUT only)

---

### **DimoMaint API Calls - Use Integration Proxy**

When calling DimoMaint APIs, always use the integration proxy with this exact format:

#### **1. List/Filter Resources (POST)**

```javascript
// Example: List work orders with filters
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkOrder/Filter',
    method: 'POST',
    body: {
      currentPage: 1,
      pageSize: 50,  // Max: 1000
      // Optional filters:
      archived: false,
      statusCode: 'IN_PROGRESS'
    }
  })
})
```

Example parameters you can use:
```json
{
  "accountAssignmentCode": "string",
  "archived": true,
  "assetCode": "string",
  "assetCriticalityCode": "string",
  "assetCriticalityName": "string",
  "assetInterfaceKey": "string",
  "bodyName": "string",
  "causeName": "string",
  "code": "string",
  "contractCode": "string",
  "currencyIsoCode": "string",
  "effectName": "string",
  "endBeginDateTime": "2025-10-30T16:00:25.586Z",
  "endCreationDateTime": "2025-10-30T16:00:25.586Z",
  "endEndDateTime": "2025-10-30T16:00:25.586Z",
  "endLastModificationDateTime": "2025-10-30T16:00:25.586Z",
  "endLimitDateTime": "2025-10-30T16:00:25.586Z",
  "issuerUserName": "string",
  "preventiveMaintenanceCode": "string",
  "remedyName": "string",
  "startBeginDateTime": "2025-10-30T16:00:25.586Z",
  "startCreationDateTime": "2025-10-30T16:00:25.586Z",
  "startEndDateTime": "2025-10-30T16:00:25.586Z",
  "startLastModificationDateTime": "2025-10-30T16:00:25.586Z",
  "startLimitDateTime": "2025-10-30T16:00:25.586Z",
  "stateName": "string",
  "subcontractorCode": "string",
  "subcontractorName": "string",
  "technicianCategoryCode": "string",
  "technicianUserName": "string",
  "technologyCode": "string",
  "title": "string",
  "urgencyCode": "string",
  "workingTypeCode": "string",
  "workRequestCode": "string",
  "currentPage": 0,
  "pageSize": 0
}
```

```javascript
// Example: List assets with filters
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/Asset/Filter',
    method: 'POST',
    body: {
      currentPage: 1,
      pageSize: 50,
      accountAssignmentCode: 'BUILDING_A'
    }
  })
})
```

#### **2. Get Single Resource (GET)**

```javascript
// Example: Get work order by code
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkOrder/Get?code=WO-12345',
    method: 'GET'
    // No body for GET requests
  })
})
```

```javascript
// Example: Get asset by interfaceKey
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/Asset/Get?interfaceKey=ASSET-001',
    method: 'GET'
  })
})
```

#### **3. Create Resource (POST)**

```javascript
// Example: Create work order
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkOrder/Post',
    method: 'POST',
    body: {
      assetCode: 'PUMP-001',
      description: 'Maintenance required',
      priority: 'HIGH',
      workType: 'CORRECTIVE'
    }
  })
})
```

```javascript
// Example: Create work request
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkRequest/Post',
    method: 'POST',
    body: {
      assetCode: 'HVAC-002',
      description: 'Air conditioning not working',
      requestedBy: 'john.doe@company.com'
    }
  })
})
```

#### **4. Update Resource (PATCH)**

```javascript
// Example: Update work order status
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkOrder/Patch',
    method: 'PATCH',
    body: {
      code: 'WO-12345',
      statusCode: 'COMPLETED',
      completionNotes: 'Issue resolved'
    }
  })
})
```

#### **5. Update Asset Meters/Measurements**

```javascript
// Example: Update asset meter reading
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/Asset/MeterUpdate',
    method: 'POST',
    body: {
      assetCode: 'TRUCK-001',
      meterType: 'HOURS',
      value: 1250.5,
      readingDate: '2025-10-29'
    }
  })
})
```

```javascript
// Example: Update asset measurement
fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/Asset/MeasurementReadingUpdate',
    method: 'POST',
    body: {
      assetCode: 'BOILER-001',
      measurementType: 'TEMPERATURE',
      value: 85.2,
      readingDate: '2025-10-29T14:30:00'
    }
  })
})
```

---

### **Key Rules:**

1. **Always use `integration: 'dimomaint'`** - Identifies the service
2. **Endpoint format: `/connector/{Resource}/{Action}`** - Never include tenant or base URL
3. **Filter endpoints use POST** - Even though they're queries, they use POST with body
4. **Query parameters in endpoint string** - For GET requests: `/connector/Asset/Get?code=XYZ`
5. **Pagination**: `page` (1-indexed) and `pageSize` (max 1000) in body
6. **No tenant in endpoint** - Proxy handles tenant automatically from stored credentials
7. **Date formats**:
   - Dates only: `"YYYY-MM-DD"` (no time/offset for WorkOrder dates)
   - Timestamps: `"YYYY-MM-DDTHH:mm:ss"`

---

### **Available Resources:**

| Resource | Filter (POST) | Get (GET) | Create (POST) | Update (PATCH) |
|----------|--------------|-----------|---------------|----------------|
| **Asset** | `/connector/Asset/Filter` | `/connector/Asset/Get` | `/connector/Asset/Post` | `/connector/Asset/Patch` |
| **WorkOrder** | `/connector/WorkOrder/Filter` | `/connector/WorkOrder/Get` | `/connector/WorkOrder/Post` | `/connector/WorkOrder/Patch` |
| **WorkRequest** | `/connector/WorkRequest/Filter` | `/connector/WorkRequest/Get` | `/connector/WorkRequest/Post` | - |
| **Part** | `/connector/Part/Filter` | `/connector/Part/Get` | - | `/connector/Part/Patch` |
| **PartStock** | `/connector/PartStock/Filter` | - | - | - |
| **User** | `/connector/User/Filter` | - | - | - |

---

### **Response Format:**

**The integration proxy wraps ALL responses in this structure:**
```javascript
{
  success: true,       // boolean
  status: 200,         // HTTP status code
  data: { /* actual DimoMaint response */ }
}
```

**DimoMaint data inside `data`:**
```javascript
// Filter endpoints: data is an array
result.data = [
  { code: 'WO-001', description: '...', statusCode: 'OPEN' },
  { code: 'WO-002', description: '...', statusCode: 'IN_PROGRESS' }
]

// Get endpoints: data is a single object
result.data = { code: 'WO-001', description: '...', statusCode: 'OPEN' }

// Create/Update: data is a message array
result.data = [
  {
    message: "Work order created successfully",
    messageCodes: ["SuccessCode"],
    messageConnectorType: "Information"  // or "Error", "Warning"
  }
]
```

**IMPORTANT:** Always access the actual DimoMaint data via `result.data`, NOT `result` directly.

**Error Handling:**
- Check `result.success` or `result.status` from the proxy wrapper
- DimoMaint errors return `messageConnectorType: "Error"` inside `result.data`
- HTTP 401: Authentication failed
- HTTP 404: Resource not found
- HTTP 406: Validation error
- HTTP 500: Server error

---

### **Common Patterns:**

```javascript
// Pattern 1: List all work orders
const result = await fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkOrder/Filter',
    method: 'POST',
    body: { currentPage: 1, pageSize: 100 }
  })
}).then(r => r.json());

if (!result.success) throw new Error(`API error: ${result.status}`);
const workOrders = result.data || [];

// Pattern 2: Get specific asset
const result = await fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/Asset/Get?code=PUMP-001',
    method: 'GET'
  })
}).then(r => r.json());

if (!result.success) throw new Error(`API error: ${result.status}`);
const asset = result.data;

// Pattern 3: Create and track work order
const result = await fetch(INTEGRATION_PROXY_URL, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    integration: 'dimomaint',
    endpoint: '/connector/WorkOrder/Post',
    method: 'POST',
    body: {
      assetCode: 'PUMP-001',
      description: 'Maintenance required',
      priority: 'HIGH'
    }
  })
}).then(r => r.json());

if (!result.success) throw new Error(`API error: ${result.status}`);

// Check DimoMaint-specific result inside data
if (result.data[0].messageConnectorType === 'Error') {
  console.error('Failed:', result.data[0].message);
} else {
  console.log('Success:', result.data[0].message);
}
```
