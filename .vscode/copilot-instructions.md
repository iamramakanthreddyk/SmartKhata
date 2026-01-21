# SmartKhata: Copilot Context
**AUTO-LOADED by VS Code for this project only**

## System Overview: Docs → Copilot → Code

```
┌─────────────────────────────────────────────────────────────┐
│                     YOU (Request)                            │
│        "Add credit limit feature to customers"              │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│              COPILOT (Reads Auto-Loaded Context)            │
│  • .vscode/copilot-instructions.md ← This file             │
│  • docs/core/01_ARCHITECTURE.md ← Current schema           │
│  • docs/core/02_FEATURES.md ← What exists                 │
│  • docs/core/03_API.md ← Endpoint patterns                │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│           COPILOT (Asks for Clarification)                 │
│  "Should credit_limit be:                                  │
│   1. Hard limit (no invoice over this amount)              │
│   2. Soft limit (warning only)                             │
│   3. Both with flag?"                                      │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                YOU (Answer)                                 │
│              "Hard limit - option 1"                        │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│         COPILOT (Updates Docs FIRST)                        │
│  ✓ 01_ARCHITECTURE.md - adds credit_limit column           │
│  ✓ 03_API.md - adds creditLimit field to response          │
│  ✓ 02_FEATURES.md - adds feature entry                     │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│         COPILOT (Implements ALL 7 Layers)                   │
│  ✓ Layer 1: Zod schema with creditLimit validation         │
│  ✓ Layer 2: UI form field for credit limit                 │
│  ✓ Layer 3: Adapter (credit_limit ↔ creditLimit)          │
│  ✓ Layer 4: Route returning creditLimit field              │
│  ✓ Layer 5: Service with credit limit logic                │
│  ✓ Layer 6: Repository querying credit_limit               │
│  ✓ Layer 7: Migration SQL adding column                    │
└────────────────────────┬────────────────────────────────────┘
                         │
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    CODE (Generated)                         │
│  All layers consistent, no gaps, ready to deploy           │
│  Docs and code always in sync ✓                            │
└─────────────────────────────────────────────────────────────┘
```

## Architecture: 7-Layer LEGO Model
Every feature must follow this structure:
1. **Types** - Zod schemas (single source of truth)
2. **Presentation** - UI with error boundaries
3. **API Client** - HTTP + name transforms
4. **REST API** - Thin routes (/api/v1, /api/v2)
5. **Services** - Business logic
6. **Repositories** - Data access + caching
7. **Database** - PostgreSQL, RLS, soft delete

## Naming Convention
- **Database**: `snake_case` (user_id, created_at, discount_percent)
- **API Response**: `camelCase` (userId, createdAt, discountPercent)
- **Frontend**: `camelCase` (userId, createdAt, discountPercent)
- **Transformation**: Adapters convert ONLY at API boundaries, nowhere else

## Critical Rules
1. **Soft Delete**: Use `is_deleted` flag, never CASCADE delete
2. **Foreign Keys**: `ON DELETE RESTRICT` (prevents cascading failures)
3. **Names**: Database ↔ API transform happens in ONE adapter file only
4. **Features**: Update existing docs in `docs/core/` (don't create new .md files)
5. **API Versions**: Support `/api/v1` AND `/api/v2` simultaneously if breaking changes needed

## Adding a Feature
1. **Discover** (3 documents - READ FIRST):
   - `docs/core/02_FEATURES.md` - Does feature exist? What's the scope?
   - `docs/core/01_ARCHITECTURE.md` - What DB tables exist? Schema relationships?
   - `docs/core/03_API.md` - What endpoints exist? What's the pattern?

2. **Update Docs BEFORE Coding** (Single Source of Truth):
   - Add feature to `02_FEATURES.md` (one sentence)
   - Update `01_ARCHITECTURE.md` (add table/column in ERD section)
   - Add endpoint to `03_API.md` (endpoint name, params, response)
   - Update `01_ARCHITECTURE.md` with migration SQL if schema changes

3. **Implement** (Following docs exactly):
   - Layer 1: Zod schema from updated types/
   - Layer 2: UI from component pattern
   - Layer 3: Adapter for name transforms
   - Layer 4: Route matching `03_API.md` spec
   - Layer 5: Service using business logic
   - Layer 6: Repository reading from updated schema
   - Layer 7: Migration file matching `01_ARCHITECTURE.md`

4. **Keep in Sync** (After code change):
   - **Changed DB column?** → Update `01_ARCHITECTURE.md` ERD immediately
   - **Changed API response?** → Update `03_API.md` endpoint spec
   - **Added feature?** → Update `02_FEATURES.md`
   - **Different logic?** → Update `ARCHITECTURE_DECISION_MATRIX.md`

## Multi-Frontend: Shared Types
All frontends (iOS/Android/Web/Windows) use **ONE Zod schema from backend**.
- Backend: `src/types/auth.ts` (single source)
- Frontend: Import type and validate locally
- NO duplicate types in frontend code

## When Stuck
- **DB Relation Issue**: Check `01_ARCHITECTURE.md` schema & ERD
- **Naming Mismatch**: Review adapter in `api/adapters/`
- **Feature Scope**: Read `02_FEATURES.md` for what exists
- **Endpoint Wrong**: Check `03_API.md` for spec
- **Sync Issue**: If code doesn't match docs, UPDATE DOCS FIRST before coding

## When Development is Done & Gaps Appear

**Scenario**: Development is complete. User says "I need customers to track discount history"

**Can we adapt easily?** YES. Here's exactly what happens:

### Step 1: You Tell Copilot the Gap
```
You: "I need to track discount changes over time - history of discounts applied"
```

### Step 2: Copilot Reads Current State
```
Copilot reads:
- 01_ARCHITECTURE.md (current schema: parties table has discount_percent)
- 02_FEATURES.md (what features exist)
- 03_API.md (current endpoints)
- HOW_TO_ADD_FEATURES.md (workflow)
```

### Step 3: Copilot Asks Questions
```
Copilot: "I found 3 options:
1. Add audit table: discount_audit (party_id, old_value, new_value, changed_at)
2. Add to parties: discount_history JSON column
3. Add to invoices: discount_applied column

Which approach?"
```

### Step 4: You Choose
```
You: "Option 1 - separate audit table, good for compliance"
```

### Step 5: Copilot Updates Docs FIRST
```
Copilot updates 01_ARCHITECTURE.md:
  discount_audit:
    id: uuid
    party_id: uuid (FK → parties.id)
    old_discount_percent: numeric(5,2)
    new_discount_percent: numeric(5,2)
    changed_by: uuid (FK → users.id)
    changed_at: timestamp
    reason: varchar(500)

Copilot updates 03_API.md:
  GET /api/v1/parties/{id}/discount-history
  Response: [
    {
      oldDiscountPercent: number
      newDiscountPercent: number
      changedBy: string
      changedAt: timestamp
      reason: string
    }
  ]
```

### Step 6: Copilot Implements in All 7 Layers
```
Layer 1: Zod schema for DiscountAudit
Layer 2: UI showing discount history table
Layer 3: Adapter to transform discount_percent ↔ discountPercent
Layer 4: New route GET /api/v1/parties/{id}/discount-history
Layer 5: Service method getDiscountHistory(partyId)
Layer 6: Repository query getting discount_audit records
Layer 7: Migration SQL creating discount_audit table
```

### Step 7: Result
- ✓ Audit table exists with correct schema
- ✓ API endpoint works and returns camelCase
- ✓ UI shows discount history
- ✓ Docs are updated (01_ARCHITECTURE.md, 03_API.md, 02_FEATURES.md)
- ✓ No cascade failures (ON DELETE RESTRICT on FK)
- ✓ Soft deletes work (is_deleted still used)
- ✓ Next person (or future you) reads docs, knows about discount_audit table

---

## Specific Example: Adding One Column

**Your exact question: "I need a new column in a table"**

Scenario: Need `credit_limit` column in `parties` table

### What Happens (4 steps):
```
Step 1: Update docs/core/01_ARCHITECTURE.md
  BEFORE:
    parties:
      id: uuid
      name: varchar
      discount_percent: numeric
      is_deleted: boolean
  
  AFTER:
    parties:
      id: uuid
      name: varchar
      discount_percent: numeric
      credit_limit: numeric(10,2) DEFAULT 0  ← ADDED
      is_deleted: boolean

Step 2: Update docs/core/03_API.md
  GET /api/v1/parties/{id}
  Response:
    {
      id, name, discountPercent,
      creditLimit,  ← ADDED (camelCase!)
      isDeleted
    }

Step 3: Tell Copilot: "Add credit_limit to parties"
  Copilot:
  - Creates migration: ALTER TABLE parties ADD COLUMN credit_limit numeric(10,2) DEFAULT 0;
  - Updates Party Zod schema to include creditLimit
  - Updates PartyAdapter to transform credit_limit → creditLimit
  - Updates PartyRepository query to SELECT credit_limit
  - Updates UI forms to show creditLimit field
  - Done in <30 seconds

Step 4: Deploy & verify
  - Migration runs
  - API returns creditLimit
  - UI shows new field
  - Next feature request? Copilot knows about creditLimit already ✓
```

## Architecture Sync: How Docs Stay Current

**Problem**: You add a DB column but forget to update docs → Copilot generates wrong code next time

**Solution - Docs-First Workflow**:
1. **Design First**: Update `01_ARCHITECTURE.md` ERD with new column
2. **API Second**: Update `03_API.md` with new request/response fields
3. **Code Third**: Write migration & code to match docs
4. **Result**: Copilot ALWAYS has current schema, never confused

**Example: Adding "discountPercent" column to parties table**:
```
Step 1 - Update docs/core/01_ARCHITECTURE.md:
  parties:
    id: uuid
    name: varchar
    phone: varchar
    email: varchar
    discount_percent: numeric(5,2)  ← NEW COLUMN
    created_at: timestamp
    is_deleted: boolean

Step 2 - Update docs/core/03_API.md:
  GET /api/v1/parties/{id}
  Response: {
    id, name, phone, email, 
    discountPercent,  ← NEW FIELD (camelCase)
    createdAt, isDeleted
  }

Step 3 - Code migration & adapter:
  - CREATE migration adding discount_percent column
  - Update PartyAdapter to transform discount_percent → discountPercent
  - Update PartyService to include discount logic
  - Update PartyRepository to query new column
  - Update Party Zod schema to validate new field

Next time Copilot helps with parties: It reads updated 01_ARCHITECTURE.md, 
knows about discountPercent, generates code correctly ✓
```

## Files You Need
- `docs/core/01_ARCHITECTURE.md` - ERD is source of truth (update this first)
- `docs/core/02_FEATURES.md` - Feature list (keep updated)
- `docs/core/03_API.md` - Endpoint reference (must match code)
- `docs/core/HOW_TO_ADD_FEATURES.md` - Detailed workflow with sync steps
- `.git/hooks/pre-commit` - Auto-prevents markdown bloat (automatic)

**DO NOT:**
- Create new .md files (git hook blocks them)
- Use CASCADE delete (causes failures)
- Add names to API without adapter transform
- Duplicate types in frontend
- Code before updating docs
- Let code drift from docs

---
**This file loads automatically in VS Code. No manual setup needed. When you open SmartKhata in another VS Code window, ONLY this context loads—no confusion with other projects.**
