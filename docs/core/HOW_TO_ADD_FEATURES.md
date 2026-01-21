# How to Add Features: The Correct Workflow

**Purpose**: When you ask Copilot to "add a new feature", this is what happens (and what SHOULD happen)  
**Audience**: Developers, Copilot, tech leads  
**Status**: CRITICAL - Use this for EVERY feature

---

## 🚀 Feature Addition Workflow (Correct Way)

### Before Coding: Discovery Phase (15 minutes)

**Your Request to Copilot**:
> "Add customer discount management feature - customers should get % discounts based on volume"

**What Copilot Should Do (NOT create new docs):**

#### Step 1: Gather Information (Check Existing Knowledge)
```
Copilot's internal checklist:
├─ Read: docs/core/02_FEATURES.md
│  └─ "Does discount feature already exist?"
│  └─ "What features are related?"
│
├─ Read: docs/core/01_ARCHITECTURE.md
│  └─ "What's the database schema?"
│  └─ "Which tables need changes?"
│
├─ Read: src/types/shared/ (if codebase exists)
│  └─ "What types exist for products, customers?"
│  └─ "Do I need to extend Party, Inventory, or Invoice types?"
│
├─ Read: docs/core/00_FOUNDATION_ARCHITECTURE.md Section LAYER 5/6/7
│  └─ "Which service should own discount logic? (PartyService? InvoiceService?)"
│  └─ "Does this need a new entity? (discounts table?)"
│
└─ Read: docs/core/ERROR_CODES.md
   └─ "What errors might this feature throw?"
```

#### Step 2: Ask Clarifying Questions
```
Copilot asks you:
"I found that discounts might go in:
1. Party table (is_preferred_customer, discount_percent)
2. New discounts table (party_id, min_quantity, discount_percent)
3. Invoice table (discount_amount already exists)

Which approach? Or should I check the existing pattern?"
```

#### Step 3: Update EXISTING Documents (Don't Create New Ones)
```
DON'T create: NEW_FEATURE_DISCOUNT_SPEC.md
DO update: docs/core/02_FEATURES.md
  └─ Add discount feature under "Features" section
  └─ Link to database schema changes

DON'T create: DISCOUNT_IMPLEMENTATION.md
DO update: docs/implementation/FOUNDATION_ARCHITECTURE_IMPLEMENTATION.md
  └─ Add as example in Phase 2: Implementation

DON'T create: DISCOUNT_TYPES.md
DO update: /src/types/shared/party.types.ts (or create party.discount.types.ts)
  └─ Add discount-related types here
```

---

## 🔍 Discovery: 3-Document Rule

**BEFORE asking "what should I do?", Copilot should check these 3 documents:**

### Document 1: Features Specification
```
File: docs/core/02_FEATURES.md

Search for: "discount" (case-insensitive)
If found: "Discount feature exists on line X"
If not found: "No discount feature yet, you're adding new feature"

Extract:
├─ Existing discount features
├─ Related features (promotions, loyalty, pricing)
└─ Any implementation notes
```

### Document 2: Database Schema
```
File: docs/core/01_ARCHITECTURE.md

Search for: Database tables
Questions Copilot should answer:
├─ "Is there a discounts table?"
├─ "Should discount data be in parties table?"
├─ "Should discount data be in invoices table?"
└─ "How are related entities connected (FKs)?"

Extract:
├─ Existing tables that could store discounts
├─ What needs modification
└─ What needs new table
```

### Document 3: Architecture Foundation
```
File: docs/core/00_FOUNDATION_ARCHITECTURE.md

Search for: Layer 5 (Services), Layer 6 (Repositories), Layer 7 (Database)
Questions Copilot should answer:
├─ "Which service should own discount logic?"
├─ "What validations are needed?"
├─ "Should discount calculation be in Service or trigger in DB?"
└─ "What business rules apply?"

Extract:
├─ Service responsibility
├─ Repository pattern to follow
└─ Database constraints needed
```

---

## 📋 The "Add Feature" Checklist (NOT create docs, UPDATE docs)

### Phase 1: Planning (Check Before Coding)

- [ ] **Check docs/core/02_FEATURES.md** - Does feature already exist?
- [ ] **Check docs/core/01_ARCHITECTURE.md** - Where in DB schema?
- [ ] **Check docs/core/00_FOUNDATION_ARCHITECTURE.md** - Which layer owns it?
- [ ] **Check /src/types/shared/** - Can I reuse existing types?
- [ ] **Check docs/core/ERROR_CODES.md** - What errors this needs?
- [ ] **Check /docs/core/ARCHITECTURE_DECISION_MATRIX.md** - Which pattern to use?

**RESULT**: You know exactly what to do, where to do it, without creating new docs

---

### Phase 2: Implementation (Follow the pattern)

#### 2a: Add to Database Schema

**File to update**: `docs/core/01_ARCHITECTURE.md` Section 🗄️ (NOT create new file)

```markdown
---
## 🗄️ Database Schema (Existing section, update here)

### New Table: discounts (Added for customer discount feature)
```sql
CREATE TABLE discounts (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  party_id UUID REFERENCES parties(id),  -- NULL = default/global
  
  min_quantity DECIMAL(12,3),            -- Min qty for discount
  discount_percent DECIMAL(5,2) NOT NULL,
  discount_type VARCHAR(50),             -- volume, loyalty, seasonal
  
  start_date DATE,
  end_date DATE,
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Link to existing invoice pricing
-- When creating invoice, calculate discount from this table
```
```

**File to update**: `docs/core/ENTITY_RELATIONSHIPS.md` (update, don't create)

```markdown
### Discount ← Party
├─ discounts.party_id → parties.id
├─ ON DELETE: SET NULL (discount applies to all if party deleted)
└─ Soft delete: Use is_active flag
```

#### 2b: Define Types

**File to create/update**: `/src/types/shared/discount.types.ts` (NOT new doc, just types)

```typescript
// Database DTO (snake_case)
export interface DiscountDbDto {
  id: string;
  business_id: string;
  party_id: string | null;
  min_quantity: number;
  discount_percent: number;
  discount_type: 'volume' | 'loyalty' | 'seasonal';
  start_date: string;
  end_date: string | null;
  is_active: boolean;
  created_at: string;
  updated_at: string;
}

// API DTO (camelCase)
export interface DiscountApiDto {
  id: string;
  businessId: string;
  partyId: string | null;
  minQuantity: number;
  discountPercent: number;
  discountType: 'volume' | 'loyalty' | 'seasonal';
  startDate: string;
  endDate: string | null;
  isActive: boolean;
  createdAt: string;
  updatedAt: string;
}

// Request DTO
export interface CreateDiscountRequest {
  partyId?: string;
  minQuantity: number;
  discountPercent: number;
  discountType: 'volume' | 'loyalty' | 'seasonal';
  startDate: string;
  endDate?: string;
}
```

#### 2c: Add Zod Schema

**File to update**: `src/types/validation/` (add discount.schema.ts, don't create new doc)

```typescript
import { z } from 'zod';

export const CreateDiscountSchema = z.object({
  partyId: z.string().uuid().optional(),
  minQuantity: z.number().positive(),
  discountPercent: z.number().min(0).max(100),
  discountType: z.enum(['volume', 'loyalty', 'seasonal']),
  startDate: z.string().date(),
  endDate: z.string().date().optional(),
});

export type CreateDiscountRequest = z.infer<typeof CreateDiscountSchema>;
```

#### 2d: Implement Service Layer

**File to create**: `src/services/DiscountService.ts` (NOT new doc, just code)

```typescript
export class DiscountService {
  constructor(
    private discountRepo: DiscountRepository,
    private partyRepo: PartyRepository
  ) {}

  /**
   * Calculate discount for a party
   * ✅ Business logic: What discount applies?
   * ❌ DON'T: Query database directly, format response
   */
  async calculateDiscount(
    partyId: string,
    quantity: number
  ): Promise<number> {
    // Get applicable discounts for party
    const discounts = await this.discountRepo.findApplicableForParty(
      partyId,
      quantity
    );

    if (discounts.length === 0) return 0;

    // Find best discount (highest %)
    return Math.max(...discounts.map(d => d.discount_percent));
  }

  /**
   * Create discount
   * ✅ Validate: Party exists, dates valid
   * ❌ DON'T: Know about HTTP responses
   */
  async createDiscount(
    businessId: string,
    request: CreateDiscountRequest
  ): Promise<DiscountDbDto> {
    // Validate party exists (if specified)
    if (request.partyId) {
      const party = await this.partyRepo.findById(request.partyId);
      if (!party) {
        throw new NotFoundError('PARTY_NOT_FOUND');
      }
    }

    // Validate date range
    if (request.endDate && request.endDate <= request.startDate) {
      throw new ValidationError('END_DATE_MUST_BE_AFTER_START_DATE');
    }

    // Create via repository
    return this.discountRepo.create({
      business_id: businessId,
      party_id: request.partyId || null,
      min_quantity: request.minQuantity,
      discount_percent: request.discountPercent,
      discount_type: request.discountType,
      start_date: request.startDate,
      end_date: request.endDate || null,
      is_active: true,
    });
  }
}
```

#### 2e: Implement Repository

**File to create**: `src/repositories/DiscountRepository.ts` (NOT new doc, just code)

```typescript
export class DiscountRepository extends ResilientRepository {
  constructor(private db: Database, private cache: CacheService) {
    super('DiscountRepository');
  }

  /**
   * Find applicable discounts for party/quantity
   * ✅ Query logic: Database access, caching, retry
   * ❌ DON'T: Business rule logic, HTTP response formatting
   */
  async findApplicableForParty(
    partyId: string,
    quantity: number
  ): Promise<DiscountDbDto[]> {
    return this.execute(async () => {
      const cacheKey = `discounts:${partyId}:applicable`;

      // Try cache first
      const cached = await this.cache.get(cacheKey);
      if (cached) return cached;

      // Query: This party's discounts OR global discounts
      const result = await this.db.query<DiscountDbDto[]>(
        `SELECT * FROM discounts 
         WHERE is_active = true
         AND (party_id = $1 OR party_id IS NULL)
         AND min_quantity <= $2
         AND start_date <= CURRENT_DATE
         AND (end_date IS NULL OR end_date >= CURRENT_DATE)`,
        [partyId, quantity]
      );

      // Cache for 1 hour
      await this.cache.set(cacheKey, result, 3600);

      return result;
    });
  }
}
```

#### 2f: Add API Endpoint

**File to update**: `src/routes/v2/discounts.ts` (NOT new doc, just routes)

```typescript
import { CreateDiscountSchema } from '@/types/validation/discount.schema';
import { transformDbToApi, transformApiToDb } from '@/api/adapters';

/**
 * POST /api/v2/discounts
 * Create new discount
 */
export const createDiscount = async (req: Request, res: Response, next: NextFunction) => {
  try {
    // 1. Validate input (API layer)
    const validated = CreateDiscountSchema.parse(req.body);

    // 2. Call service (business logic)
    const discountDb = await discountService.createDiscount(
      req.user.businessId,
      validated
    );

    // 3. Transform to API format (adapter)
    const response = transformDbToApi(discountDb);

    // 4. Send response
    res.status(201).json(response);
  } catch (error) {
    next(error);  // Error handler middleware
  }
};

/**
 * GET /api/v2/discounts/:partyId/calculate
 * Calculate discount for party & quantity
 */
export const calculateDiscount = async (req: Request, res: Response, next: NextFunction) => {
  try {
    const { partyId } = req.params;
    const { quantity } = req.query;

    const percent = await discountService.calculateDiscount(
      partyId,
      Number(quantity)
    );

    res.json({
      partyId,
      quantity: Number(quantity),
      discountPercent: percent,
    });
  } catch (error) {
    next(error);
  }
};

// Register routes
router.post('/discounts', createDiscount);
router.get('/discounts/:partyId/calculate', calculateDiscount);
```

#### 2g: Update Error Codes

**File to update**: `docs/core/ERROR_CODES.md` (add section, don't create new file)

```markdown
## Business Logic Errors (422)

### Discounts
| Code | Message | Reason |
|------|---------|--------|
| END_DATE_MUST_BE_AFTER_START_DATE | End date must be after start date | Invalid date range |
| INVALID_DISCOUNT_PERCENT | Discount must be 0-100% | Out of range |
```

#### 2h: Update Features List

**File to update**: `docs/core/02_FEATURES.md` (add feature, don't create new doc)

```markdown
### [NEW] Discount Management

**Description**: Customers get % discounts based on volume or loyalty

**Database Tables**:
- New: `discounts` table (volume, loyalty, seasonal discounts)
- Modified: None (stores discount % in discounts table)

**API Endpoints**:
- POST `/api/v2/discounts` - Create discount
- GET `/api/v2/discounts/:partyId/calculate` - Calculate applicable discount
- PUT `/api/v2/discounts/:id` - Update discount
- DELETE `/api/v2/discounts/:id` - Soft delete

**Architecture**:
- Service: DiscountService (calculate, validate)
- Repository: DiscountRepository (query, cache)
- Types: /src/types/shared/discount.types.ts

**Error Codes**: END_DATE_MUST_BE_AFTER_START_DATE, INVALID_DISCOUNT_PERCENT

**Status**: ✅ Implemented Jan 21, 2026
```

---

### Phase 3: Testing (Write Tests, NOT docs)

**File to create**: `src/routes/v2/__tests__/discounts.test.ts` (tests, not documentation)

```typescript
describe('Discount Feature', () => {
  it('should calculate discount for high-volume party', async () => {
    const discount = await discountService.calculateDiscount('party-1', 100);
    expect(discount).toBeGreaterThan(0);
  });

  it('should not apply discount below min quantity', async () => {
    const discount = await discountService.calculateDiscount('party-1', 5);
    expect(discount).toBe(0);
  });

  it('should validate end date > start date', async () => {
    expect(() =>
      discountService.createDiscount(businessId, {
        startDate: '2026-02-01',
        endDate: '2026-01-01',  // Invalid!
        minQuantity: 100,
        discountPercent: 10,
        discountType: 'volume',
      })
    ).rejects.toThrow('END_DATE_MUST_BE_AFTER_START_DATE');
  });
});
```

---

### Phase 4: Integration Check (NOT documentation)

**Copilot should verify**:

- [ ] **Types**: Can frontend import from `/src/types/shared/discount.types.ts`?
- [ ] **API**: Can API clients call `/api/v2/discounts`?
- [ ] **Naming**: All API responses in camelCase?
- [ ] **Architecture**: Service owns logic, repository owns DB, routes are thin?
- [ ] **Errors**: All error codes documented in `docs/core/ERROR_CODES.md`?
- [ ] **Features List**: Updated `docs/core/02_FEATURES.md`?
- [ ] **Tests**: All business logic has tests?

---

## ❌ What Copilot Should NOT Do

### ❌ WRONG: Create new documentation files

```
DON'T create:
❌ DISCOUNT_FEATURE_SPEC.md
❌ DISCOUNT_IMPLEMENTATION_GUIDE.md
❌ DISCOUNT_ERROR_CODES.md
❌ DISCOUNT_API.md
❌ DISCOUNT_DATABASE_SCHEMA.md

Reason: Overlapping information, hard to maintain, developers don't know where to look
```

### ❌ WRONG: Ignore existing architecture

```
DON'T do:
❌ Create discount calculation in route handler
❌ Query database directly from service
❌ Mix snake_case and camelCase in response
❌ Hard delete discounts instead of soft delete
❌ Create new error codes without documenting

Reason: Breaks established patterns
```

### ❌ WRONG: Duplicate information

```
DON'T do:
❌ Document discount feature in new file when it should be in 02_FEATURES.md
❌ Create database schema file when it should be in 01_ARCHITECTURE.md
❌ Document error codes in new file when they should be in ERROR_CODES.md

Reason: Creates single-source-of-truth problem
```

---

## ✅ The Correct Summary

**When you ask to add a Discount feature, here's what happens:**

```
1. Copilot checks 02_FEATURES.md
   └─ "Discount exists? No, adding new feature"

2. Copilot reads 01_ARCHITECTURE.md
   └─ "Where in database? New discounts table"

3. Copilot reads 00_FOUNDATION_ARCHITECTURE.md
   └─ "Which layer? Service owns logic, Repository owns DB"

4. Copilot implements WITHOUT creating new docs:
   ├─ Creates /src/types/shared/discount.types.ts (types, not doc)
   ├─ Creates src/services/DiscountService.ts (code, not doc)
   ├─ Creates src/repositories/DiscountRepository.ts (code, not doc)
   ├─ Creates src/routes/v2/discounts.ts (code, not doc)
   ├─ Creates src/__tests__/discounts.test.ts (tests, not doc)
   └─ UPDATES existing docs:
       ├─ Updates docs/core/01_ARCHITECTURE.md (add schema)
       ├─ Updates docs/core/02_FEATURES.md (add feature)
       ├─ Updates docs/core/ERROR_CODES.md (add errors)
       └─ Updates docs/core/ENTITY_RELATIONSHIPS.md (add relationships)

5. Result: Everything is consistent, maintainable, no duplicates
```

---

## 🎯 Prompts for Copilot

**→ [COPILOT_PROMPTS.md](COPILOT_PROMPTS.md)** - **Copy-paste ready prompts for all scenarios**

Includes:
1. **Simple Feature** - Single table, one service
2. **Complex Feature** - Multiple tables, multiple services
3. **Integration Feature** - External APIs
4. **Bug Fix** - Fix existing feature
5. **Multi-Platform** - iOS, Android, Web, Windows
6. **Extend Existing** - Add to current feature

Each prompt guides Copilot to:
- Read the 3 discovery documents
- Report findings
- Wait for approval
- Implement code
- Update existing docs only (no new .md files)
- **CRITICAL: Keep docs in sync as code changes**
- Verify success

**How to use:**
1. Open [COPILOT_PROMPTS.md](COPILOT_PROMPTS.md)
2. Pick your scenario
3. Copy the entire prompt block
4. Paste into VS Code Copilot Chat
5. Copilot will do discovery automatically
6. **After each code change, update docs immediately**

---

## 🔄 Keeping Docs in Sync (The Critical Part)

**The Problem**: You add a database column, but forget to update `01_ARCHITECTURE.md`. Next time Copilot helps, it doesn't know about the new column.

**The Solution**: Update docs FIRST, then code.

### When You Change the Database Schema
```
BEFORE writing migration:
  1. Update docs/core/01_ARCHITECTURE.md
     - Add new column to table definition
     - Update ERD if relationships changed
  2. Update docs/core/01_ARCHITECTURE.md
     - Add migration SQL in "Database Migrations" section
  3. Write the actual migration file
  
RESULT: Next time Copilot reads 01_ARCHITECTURE.md, it knows about changes ✓
```

### When You Change an API Endpoint
```
BEFORE changing code:
  1. Update docs/core/03_API.md
     - Change endpoint spec (request/response fields)
     - Note if breaking change (bump to /api/v2)
  2. Create migration guide if breaking
  3. Update code to match spec
  
RESULT: Copilot always reads current endpoint spec ✓
```

### When You Add/Change Feature Logic
```
BEFORE implementing:
  1. Update docs/core/02_FEATURES.md
     - Add feature or update existing entry
  2. Update docs/core/ARCHITECTURE_DECISION_MATRIX.md
     - Add why this approach (if new decision)
  3. Implement service/repository
  
RESULT: Copilot knows feature scope and design intent ✓
```

### Example: Adding discount_percent Column to parties Table

```
Step 1: Update 01_ARCHITECTURE.md (ERD Section)
---
parties:
  id: uuid (PK)
  name: varchar(255)
  phone: varchar(20)
  email: varchar(255)
  ← BEFORE: ended here
  discount_percent: numeric(5,2) DEFAULT 0  ← ADD THIS
  is_deleted: boolean
  created_at: timestamp
  updated_at: timestamp
---

Step 2: Add to 01_ARCHITECTURE.md (Migrations Section)
---
ALTER TABLE parties ADD COLUMN discount_percent numeric(5,2) DEFAULT 0;
---

Step 3: Update 03_API.md
---
GET /api/v1/parties/{id}
Response:
{
  id: uuid
  name: string
  phone: string
  email: string
  discountPercent: number  ← ADD THIS (camelCase in API!)
  createdAt: timestamp
}
---

Step 4: Update 02_FEATURES.md
---
Customer Discount Management - Customers get % discounts based on volume
---

Step 5: Code the changes
- Create migration matching step 2
- Update PartyType Zod schema to include discountPercent
- Update PartyAdapter to transform discount_percent ↔ discountPercent
- Update PartyRepository to query discount_percent
- Update PartyService with discount logic
- Update UI to display/edit discountPercent

RESULT: Docs and code are in sync. Copilot ALWAYS has current state ✓
```

---

## 📝 Update Checklist (What Gets Updated, Not Created)

When adding ANY feature:

| What | Where | Status |
|------|-------|--------|
| **Database Schema** | `docs/core/01_ARCHITECTURE.md` Sec 🗄️ | UPDATE FIRST |
| **Migration SQL** | `docs/core/01_ARCHITECTURE.md` (Migrations section) | UPDATE FIRST |
| **API Spec** | `docs/core/03_API.md` | UPDATE FIRST |
| **Feature Spec** | `docs/core/02_FEATURES.md` | UPDATE FIRST |
| **Error Codes** | `docs/core/ERROR_CODES.md` | UPDATE (if new errors) |
| **Entity Relationships** | `docs/core/ENTITY_RELATIONSHIPS.md` | UPDATE (if new relations) |
| **Types** | `/src/types/shared/` (new file OK) | CREATE CODE |
| **Service** | `/src/services/` (new file OK) | CREATE CODE |
| **Repository** | `/src/repositories/` (new file OK) | CREATE CODE |
| **Routes** | `/src/routes/v2/` (new file OK) | CREATE CODE |
| **Tests** | `/src/__tests__/` (new file OK) | CREATE CODE |

**KEY**: Docs marked "UPDATE FIRST" must be changed before you write code.

---

## 🚨 Red Flag: When Copilot Breaks the Rules

If Copilot creates:
```
❌ FEATURE_XYZ_SPEC.md
❌ FEATURE_XYZ_IMPLEMENTATION.md
❌ FEATURE_XYZ_ERRORS.md
❌ FEATURE_XYZ_DATABASE.md
❌ FEATURE_XYZ_API.md
```

**That's wrong.** It should UPDATE existing docs instead.

---

**This is the workflow. Follow it for EVERY feature.** ✅
