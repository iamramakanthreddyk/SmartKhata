# Architectural Decision Matrix: Resolve Conflicts

**Purpose**: When you have multiple options, use this matrix to decide correctly  
**Audience**: Tech leads, architects, senior developers  
**Status**: DECISION GUIDE

---

## Decision 1: "Should I Create a New Type or Reuse Existing?"

```
┌─ Question: Do you have a type for this data already?
│
├─ YES → Use existing type
│   └─ But type doesn't fit?
│       ├─ Add optional fields to existing type
│       │   OR
│       └─ Create new version of type (InvoiceApiDtoV2)
│
└─ NO → Create new type in /src/types/shared/
    └─ Remember: SNAKE_CASE for DB, CAMELCASE for API
```

### Example Flow

**Scenario**: "I need to return invoice with party details"

```typescript
// Step 1: Check existing types
import { InvoiceApiDto, PartyApiDto } from '@/types/shared';

// Step 2: Can we compose them?
interface InvoiceWithPartyDto {
  invoice: InvoiceApiDto;
  party: PartyApiDto;
}
// ✅ YES - reuse existing types, compose new one

// If that doesn't work:
interface InvoiceDetailDto extends InvoiceApiDto {
  partyDetails?: PartyApiDto;
  itemCount: number;
}
// ✅ OK - extend existing type with new fields

// Only if absolutely necessary:
interface CustomInvoiceResponse {
  // Completely new structure
}
// ❌ AVOID - creates duplication
```

---

## Decision 2: "Should This Logic Go in Service or Repository?"

```
                    ┌─ BUSINESS RULE?
                    │ (Validate, calculate, coordinate)
                    │
         YES ────→ SERVICE LAYER
         │         └─ Examples:
         │            • Validate party exists
         │            • Calculate totals
         │            • Check inventory
         │
    LAYER 5         NO
  SERVICES    ────────────────────────────┐
                                          │
         ┌─────────────────────────────────┘
         │
         ▼
    DATA ACCESS?
    (Query, cache, format for DB)
    │
    YES ────→ REPOSITORY LAYER
    │         └─ Examples:
    │            • Query database
    │            • Cache results
    │            • Retry logic
    │            • Format for DB
    │
    LAYER 6  NO
    REPOSITORIES ─────→ WRONG LAYER - rethink
```

### Example: Invoice Creation

```
Step 1: Parse request ────────→ API LAYER (Layer 4)
Step 2: Validate party exists ─→ SERVICE (Layer 5)
Step 3: Calculate totals ──────→ SERVICE (Layer 5)
Step 4: Query database ────────→ REPOSITORY (Layer 6)
Step 5: Cache result ──────────→ REPOSITORY (Layer 6)
Step 6: Transform response ────→ API LAYER (Layer 4)
```

---

## Decision 3: "Should I Hard Delete or Soft Delete?"

```
┌─ Question: Is this data business-critical?
│
├─ YES → SOFT DELETE (is_deleted, is_active flags)
│   ├─ Invoices? → SOFT DELETE
│   ├─ Parties? → SOFT DELETE
│   ├─ Payments? → SOFT DELETE
│   └─ Reason: Audit trail, data recovery
│
└─ NO → Consider carefully...
    ├─ Is it referenced by other entities? → SOFT DELETE
    ├─ Is it part of audit trail? → NEVER hard delete
    ├─ Is it temporary/session data? → Can hard delete
    └─ Generally → When in doubt, SOFT DELETE
```

### Example Matrix

| Entity | Type | Reason |
|--------|------|--------|
| invoices | Soft | Audit trail, financial records |
| parties | Soft | Referenced by many tables |
| ledger | IMMUTABLE | Never delete, append-only |
| audit_logs | IMMUTABLE | Never delete, append-only |
| invoice_items | Soft | Related to invoices |
| payments | Soft | Financial records |
| staff | Soft | Security audit trail |
| temporary_uploads | Hard | Not business-critical |
| session_tokens | Hard | Expires automatically |
| businesses | Soft | Business fundamental |

---

## Decision 4: "Should I Create a New API Version?"

```
Current endpoint: /api/v1/invoices

Question: What are you changing?

├─ ADDING optional field
│  └─ Keep v1, can add field
│     └─ Reason: Backward compatible
│
├─ ADDING required field
│  └─ Create v2
│     └─ Reason: Breaking change
│
├─ REMOVING field
│  └─ Create v2
│     └─ Reason: Breaking change
│
├─ CHANGING field type (string → number)
│  └─ Create v2
│     └─ Reason: Breaking change
│
├─ FIXING BUG (same response shape)
│  └─ Patch v1 (v1.0.1)
│     └─ Reason: Non-breaking fix
│
├─ RESTRUCTURING response object
│  └─ Create v2
│     └─ Reason: Breaking change
│
├─ CHANGING error codes
│  └─ Create v2
│     └─ Reason: Clients handle errors differently
│
└─ RENAMING field (invoice_number → invoiceNumber)
   └─ Create v2
      └─ Reason: Breaking change for JSON clients
```

### Timeline After Creating V2

```
Today
├─ Launch v2 (new version)
├─ Keep v1 running (for old clients)
├─ Update docs: v1 deprecated Dec 2026
└─ Publish migration guide
    │
6 months later
├─ Contact old clients
├─ Deadline: migrate to v2 by Dec 2026
└─ Monitor v1 usage declining
    │
Dec 2026
├─ Last support for v1
├─ Send final warning
└─ Update docs: v1 EOL
    │
Jan 2027
└─ Remove v1 code, keep only v2 and v3
```

---

## Decision 5: "Should Error Be Caught and Retried?"

```
Error occurred:

├─ TRANSIENT ERROR? (temporary, might resolve)
│  ├─ Database connection timeout
│  ├─ Network timeout
│  ├─ Rate limited (429)
│  └─ Server temporarily down (503)
│      └─ YES → RETRY with exponential backoff
│          └─ Code: CircuitBreaker.execute()
│
├─ PERMANENT ERROR? (won't help to retry)
│  ├─ Invalid input (validation error)
│  ├─ Resource not found (404)
│  ├─ Unauthorized (401)
│  └─ Conflict (409)
│      └─ NO → Throw immediately, don't retry
│          └─ Code: throw error immediately
│
└─ AMBIGUOUS? Use this:
   ├─ Can user fix it? → Throw immediately
   ├─ Is it our fault? → Throw immediately
   └─ Is it external? → Retry
```

### Retry Pattern

```typescript
// ✅ CORRECT: Retry transient errors only

async function fetchWithRetry<T>(
  fn: () => Promise<T>,
  maxRetries = 3
): Promise<T> {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      // Check if retryable
      if (!isTransientError(error)) {
        throw error;  // Don't retry permanent errors
      }

      if (attempt === maxRetries) {
        throw error;  // Last attempt failed
      }

      // Exponential backoff: 1s, 2s, 4s
      const delayMs = Math.pow(2, attempt - 1) * 1000;
      await new Promise(r => setTimeout(r, delayMs));
    }
  }
}

function isTransientError(error: any): boolean {
  // Transient: network, timeout, rate limit
  return (
    error instanceof NetworkError ||
    error instanceof TimeoutError ||
    error.statusCode === 429 ||
    error.statusCode === 503
  );
}
```

---

## Decision 6: "Should This Be Handled by API, Service, or Repository?"

```
Validation type:

├─ INPUT VALIDATION (request is invalid)
│  └─ WHERE: API LAYER (routes)
│     └─ Code: CreateInvoiceSchema.parse(req.body)
│     └─ Response: 400 Bad Request
│
├─ BUSINESS RULE VALIDATION (logic broken)
│  └─ WHERE: SERVICE LAYER
│     └─ Code: Check party exists, stock available
│     └─ Response: 422 Unprocessable Entity
│
├─ DATABASE CONSTRAINT VIOLATION
│  └─ WHERE: REPOSITORY (caught & mapped)
│     └─ Code: Catch DB error, throw business error
│     └─ Response: 409 Conflict or 400 Bad Request
│
└─ SECURITY/AUTHORIZATION
   └─ WHERE: MIDDLEWARE
      └─ Code: JWT verification, RLS policy
      └─ Response: 401 Unauthorized or 403 Forbidden
```

### Example: Create Invoice

```typescript
// ===== API LAYER (validate format)
app.post('/api/v2/invoices', (req, res) => {
  // 1. Validate schema (is it even valid JSON?)
  const validated = CreateInvoiceSchema.parse(req.body);
  // Returns 400 if fails
});

// ===== SERVICE LAYER (validate business rules)
async function createInvoice(request: CreateInvoiceRequest) {
  // 2. Validate party exists (business logic)
  const party = await partyRepo.findById(request.partyId);
  if (!party) throw new NotFoundError('PARTY_NOT_FOUND');
  // Returns 404 if fails

  // 3. Validate inventory (business logic)
  if (!hasEnoughStock(request.items)) {
    throw new ValidationError('INSUFFICIENT_STOCK');
  }
  // Returns 422 if fails
}

// ===== REPOSITORY LAYER (handle DB constraints)
async function create(data: InvoiceCreateDto) {
  try {
    return await db.insert('invoices', data);
  } catch (error) {
    if (error.code === 'UNIQUE_VIOLATION') {
      throw new ConflictError('DUPLICATE_INVOICE');
    }
    throw error;
  }
}
```

---

## Decision 7: "Where Should Cache Go?"

```
Need to cache data?

├─ HTTP CLIENT LEVEL (frontend)
│  └─ Cache API responses
│  └─ TTL: Minutes (1-5 min)
│  └─ Tool: Redux, MobX, Context
│  └─ Invalidate: User action, error
│
├─ REPOSITORY LEVEL (backend)
│  └─ Cache DB query results
│  └─ TTL: Minutes to Hours (5min-1hr)
│  └─ Tool: Redis
│  └─ Invalidate: Write operations
│
├─ DATABASE LEVEL
│  └─ Cache query results
│  └─ TTL: Depends on data freshness needs
│  └─ Tool: PostgreSQL query plan cache
│  └─ Automatic (built-in)
│
└─ LOCAL STORAGE (mobile)
   └─ Cache last fetch
   └─ TTL: Hours to Days
   └─ Tool: SQLite, AsyncStorage
   └─ Fallback: When offline
```

### Cache Invalidation Rules

```typescript
// When to clear cache?

CREATE → Invalidate listing cache
  Example: Create invoice → Clear invoices_list cache

UPDATE → Invalidate specific + listing cache
  Example: Update invoice → Clear invoice:${id} + invoices_list

DELETE → Invalidate specific + listing cache
  Example: Delete invoice → Clear invoice:${id} + invoices_list

QUERY → Cache with TTL, let it expire
  Example: Get party balance → Cache 5 min, auto-expire
```

---

## Decision 8: "Should I Log This?"

```
Event occurred:

├─ ERROR or EXCEPTION
│  └─ YES → Always log
│     └─ Level: ERROR or WARN
│     └─ Include: Stack trace, context
│
├─ SECURITY EVENT
│  ├─ Login attempt? → YES, log
│  ├─ Permission denied? → YES, log
│  ├─ Data access? → YES, audit log
│  └─ Level: INFO (important)
│
├─ OPERATIONAL (info)
│  ├─ User action? → Maybe (too verbose if logging all)
│  ├─ API response time slow? → YES
│  ├─ Database query slow? → YES
│  └─ Level: DEBUG or INFO
│
├─ DEBUG
│  ├─ Function entry/exit? → NO (too verbose)
│  ├─ Variable values? → YES, use DEBUG level
│  ├─ State changes? → YES, use DEBUG level
│  └─ Level: DEBUG (dev only)
│
└─ NEVER LOG
   ├─ Passwords or secrets
   ├─ Credit card numbers
   ├─ Personal information (names in URLs)
   └─ Every single API call (enable with flag)
```

### Log Levels Quick Reference

```
TRACE   - Very detailed (function calls, values)
DEBUG   - Development info (variable states, calculations)
INFO    - Operational events (user actions, milestones)
WARN    - Warning (cache miss, retry attempt, slow operation)
ERROR   - Error occurred (exception, failed operation)
FATAL   - System failure (database down, service crashed)
```

---

## Decision 9: "Should This Be in Database or Cache?"

```
Data to store:

├─ IMMUTABLE (never changes)
│  └─ Ledger entries, audit logs
│  └─ Storage: DATABASE only
│  └─ Why: Truth source, must persist
│
├─ FREQUENTLY READ, RARELY WRITTEN
│  ├─ Party list, inventory, settings
│  └─ Storage: DATABASE + CACHE
│  └─ Why: Speed up reads, but truth in DB
│
├─ SESSION DATA (temporary)
│  ├─ Logged-in user, shopping cart
│  └─ Storage: CACHE + optionally DB
│  └─ Why: Fast, expires naturally
│
├─ TEMPORARY (worker jobs, tokens)
│  ├─ Email verification tokens, job queue
│  └─ Storage: CACHE (auto-expire)
│  └─ Why: Not needed long-term
│
└─ REAL-TIME (constantly changing)
   ├─ WebSocket connections, live counts
   └─ Storage: CACHE only
   └─ Why: Accurate as of this second
```

---

## Decision 10: "Should I Use Repository Pattern or Direct Queries?"

```
Getting data?

└─ ALWAYS use Repository Pattern

┌─ Repository = Consistent interface for data access
│
├─ BENEFITS:
│  ├─ Single place to change queries
│  ├─ Caching logic centralized
│  ├─ Retry/circuit breaker in one place
│  ├─ Easy to mock in tests
│  ├─ Can swap DB engine (DB → API, DB → cache)
│  └─ Consistent error handling
│
└─ PATTERN:
   interface Repository {
     findById(id) → returns Type | throws Error
     create(data) → returns Type | throws Error
     update(id, data) → returns Type | throws Error
     delete(id) → returns void | throws Error
   }
```

### Example: Don't Do This

```typescript
// ❌ WRONG: Direct queries scattered around
async function getInvoice(id: string) {
  const result = await db.query('SELECT * FROM invoices WHERE id = $1', [id]);
  return result;
}

async function getInvoiceAgain(id: string) {
  const result = await db.query('SELECT * FROM invoices WHERE id = $1', [id]);
  // Same query, different place!
  return result;
}

// Later: Need to add caching...
// Update in 10 different places!
```

```typescript
// ✅ CORRECT: Repository pattern
class InvoiceRepository {
  async findById(id: string) {
    // Query once, cache, retry - all handled here
    const cached = await cache.get(`invoice:${id}`);
    if (cached) return cached;

    const result = await this.execute(() =>
      db.query('SELECT * FROM invoices WHERE id = $1', [id])
    );

    await cache.set(`invoice:${id}`, result);
    return result;
  }
}

// Later: Need to add caching...
// Update in ONE place! All code automatically gets caching!
```

---

## Decision Matrix Summary Table

| Decision | If YES | If NO | Reference |
|----------|--------|-------|-----------|
| Reuse existing type? | Use existing | Create new in /src/types/ | Decision 1 |
| Is it business logic? | → Service Layer | → Repository Layer | Decision 2 |
| Hard delete safe? | Soft delete anyway | Soft delete | Decision 3 |
| Breaking change? | New API version | Keep same version | Decision 4 |
| Transient error? | Retry with backoff | Throw immediately | Decision 5 |
| Where validate? | Check each layer | Don't mix layers | Decision 6 |
| Cache helpful? | Repository cache | Let expire naturally | Decision 7 |
| Should log? | Pick right level | Don't log sensitive data | Decision 8 |
| Store in DB? | Database + cache | Cache with TTL | Decision 9 |
| Query scattered? | Use Repository | Consolidate pattern | Decision 10 |

---

## Using This Matrix

**Scenario**: "I need to add caching to GetInvoiceList endpoint"

```
Step 1: Find relevant decision
→ Decision 7: "Where Should Cache Go?"
→ Repository level caching

Step 2: Check decision tree
→ Cache DB query results
→ TTL: Minutes (5min-1hr)
→ Tool: Redis

Step 3: Implement pattern
```typescript
class InvoiceRepository {
  async list(businessId: string) {
    const cacheKey = `invoices:${businessId}`;
    
    // Check cache first
    const cached = await redis.get(cacheKey);
    if (cached) return cached;
    
    // Query DB
    const result = await db.query(
      'SELECT * FROM invoices WHERE business_id = $1',
      [businessId]
    );
    
    // Store in cache (5 min TTL)
    await redis.setex(cacheKey, 300, JSON.stringify(result));
    
    return result;
  }
}
```

Step 4: Invalidate cache on writes
```typescript
async createInvoice(data) {
  const result = await db.insert('invoices', data);
  
  // Clear the list cache
  await redis.del(`invoices:${data.business_id}`);
  
  return result;
}
```

---

**Status**: ✅ Complete decision matrix  
**Last Updated**: Jan 21, 2026
