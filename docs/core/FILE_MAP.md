# How Copilot Finds Everything: File Map

**Purpose**: Explicit file paths so Copilot knows exactly where to read  
**Audience**: Copilot, developers, AI systems  
**Status**: CRITICAL - Reference this when Copilot is confused about file locations

---

## 🗺️ File Paths: Where to Find What

### DISCOVERY PHASE: The 3 Documents

```
QUESTION: "Does this feature already exist?"
FILE:     docs/core/02_FEATURES.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\02_FEATURES.md
WHAT:     Complete feature list (38 features)
ACTION:   Search for feature name

---

QUESTION: "Where should this go in the database?"
FILE:     docs/core/01_ARCHITECTURE.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\01_ARCHITECTURE.md
WHAT:     Database schema, entity relationships
ACTION:   Find "🗄️ Database Schema" section, look for relevant tables

---

QUESTION: "Which layer owns this logic?"
FILE:     docs/core/00_FOUNDATION_ARCHITECTURE.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\00_FOUNDATION_ARCHITECTURE.md
WHAT:     7-layer architecture, responsibilities per layer
ACTION:   Read each layer, find where logic belongs
```

---

## 📚 REFERENCE PHASE: Other Essential Docs

```
QUESTION: "What error codes exist?"
FILE:     docs/core/ERROR_CODES.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\ERROR_CODES.md
WHAT:     Registry of all error codes (DISCOUNT_NOT_FOUND, PARTY_INACTIVE, etc.)
ACTION:   Search or add new codes

---

QUESTION: "What relationships exist between entities?"
FILE:     docs/core/ENTITY_RELATIONSHIPS.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\ENTITY_RELATIONSHIPS.md
WHAT:     Foreign key relationships, soft delete rules, constraints
ACTION:   Verify FK relationships before adding new ones

---

QUESTION: "What's the naming convention?"
FILE:     docs/core/00_FOUNDATION_ARCHITECTURE.md (Section: Naming Conventions)
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\00_FOUNDATION_ARCHITECTURE.md
WHAT:     Database = snake_case, API/Frontend = camelCase
ACTION:   Apply conversion rules at adapter layer

---

QUESTION: "How should I handle this architectural decision?"
FILE:     docs/core/ARCHITECTURE_DECISION_MATRIX.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\ARCHITECTURE_DECISION_MATRIX.md
WHAT:     10 design decisions with decision trees
ACTION:   Search for your decision type

---

QUESTION: "What's the feature addition workflow?"
FILE:     docs/core/HOW_TO_ADD_FEATURES.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\HOW_TO_ADD_FEATURES.md
WHAT:     Detailed 4-phase workflow
ACTION:   Follow exact steps in this doc

---

QUESTION: "What prompts do I use for Copilot?"
FILE:     docs/core/COPILOT_PROMPTS.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\COPILOT_PROMPTS.md
WHAT:     6 scenario-specific copy-paste prompts
ACTION:   Copy your scenario, paste into Copilot Chat

---

QUESTION: "What's the quick reference cheat sheet?"
FILE:     docs/reference/QUICK_REFERENCE_FOUNDATION.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\reference\QUICK_REFERENCE_FOUNDATION.md
WHAT:     At-desk cheat sheet (1-2 pages)
ACTION:   Print and keep at desk while coding

---

QUESTION: "How do I support multi-platform (iOS, Android, Web, Windows)?"
FILE:     docs/core/MULTI_FRONTEND_ARCHITECTURE.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\MULTI_FRONTEND_ARCHITECTURE.md
WHAT:     Shared types pattern, type sharing examples
ACTION:   Use for any multi-platform feature

---

QUESTION: "How is the API versioned?"
FILE:     docs/core/API_VERSIONS.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\API_VERSIONS.md
WHAT:     Versioning strategy (/v1, /v2), breaking changes
ACTION:   Decide which version to implement in

---

QUESTION: "How do I debug this problem?"
FILE:     docs/core/TROUBLESHOOTING.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\TROUBLESHOOTING.md
WHAT:     8 architectural problems + solutions
ACTION:   Search for your problem, follow solution

---

QUESTION: "What are the API endpoints?"
FILE:     docs/core/03_API.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\03_API.md
WHAT:     All REST endpoints, request/response formats
ACTION:   Add new endpoints here after implementation

---

QUESTION: "What security requirements apply?"
FILE:     docs/core/04_SECURITY.md
PATH:     c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\04_SECURITY.md
WHAT:     Auth, JWT tokens, RLS policies, encryption
ACTION:   Check requirements before implementing
```

---

## 💻 CODE PHASE: Where to Create Code

```
NEW TYPES:
  Location: /src/types/shared/
  Example:  /src/types/shared/discount.types.ts
  Pattern:  export interface DiscountRequest { ... }

BUSINESS LOGIC:
  Location: /src/services/
  Example:  /src/services/DiscountService.ts
  Pattern:  class DiscountService { }

DATABASE ACCESS:
  Location: /src/repositories/
  Example:  /src/repositories/DiscountRepository.ts
  Pattern:  class DiscountRepository { }

HTTP ROUTES:
  Location: /src/routes/
  Pattern:  /v1/, /v2/ subdirectories by API version
  Example:  /src/routes/v2/discounts.ts

TESTS:
  Location: /src/__tests__/
  Example:  /src/__tests__/discounts.test.ts

TYPE ADAPTERS (snake_case ↔ camelCase):
  Location: /src/adapters/
  Example:  /src/adapters/discount.adapter.ts
  Pattern:  export const toDiscountApiDto = (db) => ({ ... })
```

---

## 📝 DOCUMENTATION PHASE: Which Docs to Update

**DO UPDATE (Always):**
```
1. docs/core/02_FEATURES.md
   └─ Add your feature to the features list

2. docs/core/01_ARCHITECTURE.md
   └─ Add/modify database schema in "🗄️ Database Schema" section

3. docs/core/ERROR_CODES.md
   └─ Add error codes if feature throws errors

4. docs/core/ENTITY_RELATIONSHIPS.md
   └─ Add entity relationships if new tables/FKs
```

**DO NOT CREATE (Never):**
```
❌ FEATURE_DISCOUNT_SPEC.md
❌ DISCOUNT_IMPLEMENTATION_GUIDE.md
❌ DISCOUNT_API_REFERENCE.md
❌ DISCOUNT_DATABASE_SCHEMA.md
❌ MY_FEATURE_ERRORS.md
❌ FEATURE_XYZ_TYPES.md
```

---

## 🔄 Complete Discovery Sequence

**When adding a feature, read in this order:**

```
START: Feature request comes in
       Example: "Add customer discount management"

STEP 1: Read discovery documents
        ├─ Open: docs/core/02_FEATURES.md
        │  └─ Search: "discount" → "Not found, new feature"
        ├─ Open: docs/core/01_ARCHITECTURE.md
        │  └─ Find section: "🗄️ Database Schema"
        │  └─ Identify: parties, invoices tables
        └─ Open: docs/core/00_FOUNDATION_ARCHITECTURE.md
           └─ Find: "Layer 5: Business Logic Services"
           └─ Identify: InvoiceService might own this

STEP 2: Check supplementary docs
        ├─ Open: /src/types/shared/
        │  └─ Check: party.types.ts, invoice.types.ts
        ├─ Open: docs/core/ERROR_CODES.md
        │  └─ List: What errors needed? (DISCOUNT_NOT_FOUND?)
        └─ Open: docs/core/ENTITY_RELATIONSHIPS.md
           └─ Verify: How relate discount → party?

STEP 3: Report findings
        "Feature: NEW
         Table: discounts (party_id, discount_percent)
         Service: InvoiceService
         Types: Extend Party, create DiscountRequest"

STEP 4: Get approval, implement, update docs
        ├─ Implement code
        ├─ Update 02_FEATURES.md
        ├─ Update 01_ARCHITECTURE.md
        ├─ Update ERROR_CODES.md
        └─ Update ENTITY_RELATIONSHIPS.md

END: Feature complete, documented, ready to merge
```

---

## 🎯 Decision Tree: "Where do I find X?"

```
"I need to know if this feature exists"
  → Open docs/core/02_FEATURES.md
  → Search: feature name
  → If found: Read description
  → If not found: You're adding new feature

"I need to know database structure"
  → Open docs/core/01_ARCHITECTURE.md
  → Find: "🗄️ Database Schema" section
  → Locate: Table you need
  → Check: Columns and relationships

"I need to know what layer owns this"
  → Open docs/core/00_FOUNDATION_ARCHITECTURE.md
  → Read: "Layer 5: Business Logic Services"
  → Match: Your logic to appropriate service
  → Read: "Layer 6: Data Access" for repository

"I need to know error codes"
  → Open docs/core/ERROR_CODES.md
  → Search: Related error codes
  → Add: New codes needed for your feature

"I need an example of how to implement this"
  → Open docs/core/HOW_TO_ADD_FEATURES.md
  → Find: "Discount Feature Example"
  → Follow: Complete implementation walkthrough

"I need the prompt to give Copilot"
  → Open docs/core/COPILOT_PROMPTS.md
  → Pick: Your scenario (Simple, Complex, etc.)
  → Copy: Entire prompt
  → Paste: Into VS Code Copilot Chat

"I need quick reference at my desk"
  → Open docs/reference/QUICK_REFERENCE_FOUNDATION.md
  → Print: 1-2 pages
  → Keep: Next to keyboard while coding

"I'm stuck on an architectural decision"
  → Open docs/core/ARCHITECTURE_DECISION_MATRIX.md
  → Find: Decision tree for your situation
  → Read: Each option and consequence

"I need multi-platform (iOS/Android/Web/Windows) support"
  → Open docs/core/MULTI_FRONTEND_ARCHITECTURE.md
  → Find: Payment status example
  → Copy: Pattern for your feature

"I'm debugging a problem"
  → Open docs/core/TROUBLESHOOTING.md
  → Find: Your problem description
  → Follow: Solution steps provided
```

---

## 📍 Root Directory Structure

```
SmartKhata/
├── README.md
│   └─ START HERE (links to COPILOT_PROMPTS.md)
│
├── ARCHITECTURE_FOUNDATION_COMPLETE.md
│   └─ Overview of everything created
│
├── docs/
│   ├── core/
│   │   ├── 00_FOUNDATION_ARCHITECTURE.md ⭐ Layer definitions
│   │   ├── 01_ARCHITECTURE.md ⭐ Database schema
│   │   ├── 02_FEATURES.md ⭐ Feature list
│   │   ├── 03_API.md - Endpoints
│   │   ├── 04_SECURITY.md - Auth & compliance
│   │   ├── ERROR_CODES.md ⭐ Error registry
│   │   ├── ENTITY_RELATIONSHIPS.md ⭐ FK relationships
│   │   ├── HOW_TO_ADD_FEATURES.md ⭐ Workflow
│   │   ├── COPILOT_PROMPTS.md ⭐ Copy-paste prompts
│   │   ├── ARCHITECTURE_DECISION_MATRIX.md - Decisions
│   │   ├── API_VERSIONS.md - Versioning
│   │   ├── MULTI_FRONTEND_ARCHITECTURE.md - iOS/Android/Web
│   │   ├── TROUBLESHOOTING.md - Problems + solutions
│   │   └── [5 more docs...]
│   │
│   ├── reference/
│   │   ├── QUICK_REFERENCE_FOUNDATION.md ⭐ Cheat sheet
│   │   └── [other indexes...]
│   │
│   ├── implementation/
│   └── strategic/
│
└── /src/ (code location)
    ├── types/shared/ ← Put types here
    ├── services/ ← Put business logic here
    ├── repositories/ ← Put DB access here
    ├── routes/ ← Put HTTP routes here
    ├── adapters/ ← Put transformers here
    └── __tests__/ ← Put tests here
```

---

## ✅ Verification: Did Copilot Find Everything?

When Copilot starts, it should say something like:

```
"I'm reading SmartKhata architecture...

1. ✓ Found docs/core/02_FEATURES.md - 38 existing features
2. ✓ Found docs/core/01_ARCHITECTURE.md - Database schema
3. ✓ Found docs/core/00_FOUNDATION_ARCHITECTURE.md - 7-layer model
4. ✓ Found /src/types/shared/ - Existing type patterns
5. ✓ Found docs/core/ERROR_CODES.md - 40 error codes
6. ✓ Found docs/core/ENTITY_RELATIONSHIPS.md - Entity links

Ready to analyze your request..."
```

If Copilot can't find files:
- Use absolute paths from "CODE PHASE" section above
- Paste file path into Copilot Chat
- Say: "These files are at: [paste path]"

---

## 🔗 Quick Links to Key Files

| Need | File | Path |
|------|------|------|
| Feature exists? | 02_FEATURES.md | docs/core/ |
| DB structure? | 01_ARCHITECTURE.md | docs/core/ |
| Which layer? | 00_FOUNDATION_ARCHITECTURE.md | docs/core/ |
| Error codes? | ERROR_CODES.md | docs/core/ |
| Relationships? | ENTITY_RELATIONSHIPS.md | docs/core/ |
| How-to guide? | HOW_TO_ADD_FEATURES.md | docs/core/ |
| Copilot prompts? | COPILOT_PROMPTS.md | docs/core/ |
| Cheat sheet? | QUICK_REFERENCE_FOUNDATION.md | docs/reference/ |
| Multi-platform? | MULTI_FRONTEND_ARCHITECTURE.md | docs/core/ |
| Problem help? | TROUBLESHOOTING.md | docs/core/ |

---

## 💡 Example: Copilot Using This Map

```
DEVELOPER: [Pastes "Simple Feature" prompt from COPILOT_PROMPTS.md]
           "Add discount feature"

COPILOT:   "Reading discovery docs from file map...
           
           1. Checking: docs/core/02_FEATURES.md
              → Searched for 'discount'
              → Not found, new feature
           
           2. Checking: docs/core/01_ARCHITECTURE.md
              → Found section: 🗄️ Database Schema
              → Found tables: parties, invoices, products
              → Decision: Add 'discounts' table
           
           3. Checking: docs/core/00_FOUNDATION_ARCHITECTURE.md
              → Found Layer 5: Services
              → Found: InvoiceService, PartyService
              → Decision: Extend InvoiceService
           
           4. Checking: /src/types/shared/
              → Found: party.types.ts, invoice.types.ts
              → Reuse: Party interface
              → Create: DiscountRequest interface
           
           5. Checking: docs/core/ERROR_CODES.md
              → Found patterns: PARTY_NOT_FOUND, INVOICE_NOT_FOUND
              → Add: DISCOUNT_NOT_FOUND, DISCOUNT_INACTIVE"
           
           FINDINGS:
           - Feature exists? NO (new)
           - DB: Add discounts table
           - Service: InvoiceService
           - Types: Reuse Party, create DiscountRequest
           - Errors: Add 2 error codes
           
           Ready to implement? YES/NO?"

DEVELOPER: "YES"

COPILOT:   "Implementing...
           - Created: /src/types/shared/discount.types.ts
           - Modified: /src/services/InvoiceService.ts
           - Created: /src/repositories/DiscountRepository.ts
           - Created: /src/routes/v2/discounts.ts
           - Created: /src/__tests__/discounts.test.ts
           
           Updating docs...
           - Updated: docs/core/01_ARCHITECTURE.md (added schema)
           - Updated: docs/core/02_FEATURES.md (added feature)
           - Updated: docs/core/ERROR_CODES.md (added 2 errors)
           - Updated: docs/core/ENTITY_RELATIONSHIPS.md (added FK)
           
           ✓ Complete, ready to merge"
```

---

**Status**: ✅ Complete  
**Next**: Use COPILOT_PROMPTS.md to add your feature
