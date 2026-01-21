# Copilot Prompts: Feature Addition Workflow

**Purpose**: Copy-paste prompts that guide Copilot through discovery → implementation → doc updates  
**Audience**: Developers who ask Copilot to "add a feature"  
**Location**: Use these prompts in VS Code Chat or GitHub Copilot Chat  

---

## 🚀 Quick Start

**Choose your scenario:**
1. [Simple Feature](#simple-feature) - One table, one service (e.g., discount %)
2. [Complex Feature](#complex-feature) - Multiple tables, multiple services
3. [Integration Feature](#integration-feature) - Connects multiple existing features
4. [Bug Fix](#bug-fix) - Fix existing feature

**Then copy the entire prompt below and paste into Copilot Chat**

---

## 📋 Simple Feature

**Use this for**: Discount %, customer status, loyalty points - features that add ONE new field/table

### Prompt (Copy-Paste This)

```
Add {FEATURE_NAME} feature to SmartKhata

DISCOVERY PHASE (check these docs first, DO NOT code yet):
1. Open docs/core/02_FEATURES.md
   - Search for "{FEATURE_NAME}" 
   - Report: "Feature exists" or "Feature doesn't exist yet"

2. Open docs/core/01_ARCHITECTURE.md
   - Find "🗄️ Database Schema" section
   - Identify: Which table should store {FEATURE_NAME}?
   - Report the table name and columns needed

3. Open docs/core/00_FOUNDATION_ARCHITECTURE.md
   - Find "Layer 5: Business Logic Services"
   - Identify: Which Service owns this logic? (PartyService? InvoiceService?)
   - Report the service name

4. Check if types exist:
   - Look for existing types in /src/types/shared/
   - Report which types can be reused

5. Check error codes:
   - Open docs/core/ERROR_CODES.md
   - Report what error codes this feature needs

AFTER discovery, ask me to confirm before implementing.

Show findings in this format:
- Feature exists? YES/NO
- Database table: [table_name]
- Columns needed: [list]
- Service: [service_name]
- Reusable types: [list]
- New error codes: [list]
```

---

## 🔧 Complex Feature

**Use this for**: Multiple tables, workflows, external integrations, state machines

### Prompt (Copy-Paste This)

```
Add {FEATURE_NAME} feature to SmartKhata

STEP 1: DISCOVERY (Read existing docs, report findings)
Copilot: Open these files and report what you find:

A) docs/core/02_FEATURES.md
   ├─ Search for "{FEATURE_NAME}"
   ├─ Search for related features: [list related words]
   └─ Report: "Feature exists" or "New feature"

B) docs/core/01_ARCHITECTURE.md
   ├─ Which tables are involved? (search for table names)
   ├─ Are there existing relationships? (search for FK)
   └─ Report: Schema changes needed

C) docs/core/00_FOUNDATION_ARCHITECTURE.md
   ├─ Layer 5: Which services exist?
   ├─ Layer 6: Which repositories exist?
   ├─ Layer 7: Do we need new tables?
   └─ Report: Service + Repository layers needed

D) Check existing patterns:
   ├─ /src/types/shared/ - What types exist?
   ├─ /src/services/ - Which services have similar logic?
   ├─ /src/repositories/ - Which repos handle related data?
   └─ Report: Patterns to follow

E) docs/core/ARCHITECTURE_DECISION_MATRIX.md
   ├─ Should this be sync or async?
   ├─ Should this trigger notifications?
   ├─ Should this be versioned?
   └─ Report: Architectural decisions

STEP 2: CONFIRM APPROACH
After discovering, show:
- Feature exists? YES/NO
- Tables involved: [list]
- Services involved: [list]
- Repositories involved: [list]
- Type definitions needed: [list]
- Decisions needed: [list questions for me]

Ask: "Should I proceed with this approach?" WAIT for my approval.

STEP 3: IMPLEMENTATION (only after approval)
- Create code files (no .md files)
- Use existing patterns from similar features
- Add error handling

STEP 4: UPDATE EXISTING DOCS (critical!)
- docs/core/01_ARCHITECTURE.md - Add schema changes
- docs/core/02_FEATURES.md - Add feature description
- docs/core/ERROR_CODES.md - Add error codes
- docs/core/ENTITY_RELATIONSHIPS.md - Add FK relationships

NEVER create new .md files for this feature.
```

---

## 🔌 Integration Feature

**Use this for**: Connect payment gateway, sync external data, link multiple systems

### Prompt (Copy-Paste This)

```
Add {FEATURE_NAME} integration to SmartKhata

PHASE 1: UNDERSTAND EXISTING INTEGRATIONS
1. Read: docs/core/09_INTEGRATIONS.md
   - What integrations exist?
   - What patterns do they follow?
   - Report: Integration pattern to follow

2. Read: docs/core/03_API.md
   - Where is the API client code?
   - What error handling exists?
   - Report: Error handling pattern

3. Check code structure:
   - /src/integrations/ - Existing integration code?
   - /src/adapters/ - Type transformation code?
   - /src/services/ - Service layer that uses integrations?
   - Report: Code organization pattern

PHASE 2: DISCOVERY (before coding)
4. Read: docs/core/02_FEATURES.md
   - Does this integration already exist?
   - Are there partial implementations?
   - Report: Current state

5. Read: docs/core/01_ARCHITECTURE.md
   - Where should integration data be stored?
   - What tables need changes?
   - Report: Schema changes

6. Read: docs/core/00_FOUNDATION_ARCHITECTURE.md
   - Layer 3: Where should API client go?
   - Layer 4: Where should routes go?
   - Layer 5: Which service coordinates?
   - Report: Layers involved

7. Read: docs/core/04_SECURITY.md
   - How to store API keys securely?
   - What auth does external API need?
   - Report: Security requirements

PHASE 3: CONFIRM & IMPLEMENT
- Show all findings
- Ask for confirmation
- Implement code (services, repos, routes, adapters)
- Add tests

PHASE 4: UPDATE EXISTING DOCS
- docs/core/01_ARCHITECTURE.md - Add schema
- docs/core/02_FEATURES.md - Add feature
- docs/core/09_INTEGRATIONS.md - Add integration details
- docs/core/03_API.md - Add new endpoints
- docs/core/04_SECURITY.md - Add security requirements
- docs/core/ERROR_CODES.md - Add error codes

NEVER create new integration docs.
```

---

## 🐛 Bug Fix

**Use this for**: Fix existing feature without changing DB or architecture

### Prompt (Copy-Paste This)

```
Fix {BUG_DESCRIPTION} in SmartKhata

ANALYSIS PHASE:
1. Read: docs/core/02_FEATURES.md
   - Find the feature that has this bug
   - Report: Feature description and current behavior

2. Read: docs/core/01_ARCHITECTURE.md
   - Identify which layer has the bug
   - Report: Layer and component involved

3. Read: docs/core/TROUBLESHOOTING.md
   - Is this a known issue?
   - Report: Known workarounds

4. Check code:
   - Locate the buggy component
   - Understand current implementation
   - Report: Root cause hypothesis

5. Read: docs/core/ERROR_CODES.md
   - What error codes relate to this?
   - Report: Error handling currently in place

FIX PHASE:
- Propose fix without database changes
- Propose tests to verify fix
- Ask for confirmation before implementing

VALIDATION PHASE:
- Implement fix
- Add/update tests
- Verify no cascade failures

UPDATE DOCS:
- docs/core/02_FEATURES.md - Update if behavior changes
- docs/core/TROUBLESHOOTING.md - Document the fix if others hit it
- No new .md files

NEVER change database schema for bug fixes.
```

---

## 📊 Multi-Platform Feature (iOS, Android, Web, Windows)

**Use this for**: Features that need to work on all platforms

### Prompt (Copy-Paste This)

```
Add {FEATURE_NAME} to ALL platforms (iOS, Android, Web, Windows)

PHASE 1: SHARED TYPES
1. Read: docs/core/MULTI_FRONTEND_ARCHITECTURE.md
   - How are types shared?
   - Report: Monorepo or npm package approach

2. Check: /src/types/shared/ or @smartkhata/types
   - What types exist?
   - Can we extend existing types?
   - Report: Type structure

BEFORE CODING:
- If adding field: What's the TypeScript interface?
- If adding status: What are the enum values?
- Report: Complete type definition needed

PHASE 2: BACKEND IMPLEMENTATION
- Implement feature (Service → Repository → DB)
- Update types in /src/types/shared/
- Test with API

PHASE 3: FRONTEND IMPLEMENTATIONS
FOR EACH: iOS, Android, Web, Windows
- Import types from @smartkhata/types
- Implement UI using those types
- Test locally

PHASE 4: VERIFY ALL PLATFORMS
- Test payment on iOS
- Test payment on Android
- Test payment on Web
- Test payment on Windows
- Confirm all show same data

PHASE 5: UPDATE EXISTING DOCS
- docs/core/MULTI_FRONTEND_ARCHITECTURE.md - Add feature example
- docs/core/02_FEATURES.md - Add feature
- docs/core/03_API.md - Add endpoint
- docs/core/ERROR_CODES.md - Add errors

NEVER create separate docs per platform.
```

---

## ⚙️ Extending Existing Feature

**Use this for**: Add to an existing feature (e.g., add new discount type to discounts)

### Prompt (Copy-Paste This)

```
Extend {EXISTING_FEATURE} with {NEW_CAPABILITY}

PHASE 1: UNDERSTAND CURRENT IMPLEMENTATION
1. Read: docs/core/02_FEATURES.md
   - Find {EXISTING_FEATURE} section
   - Report: Current capabilities and limitations

2. Read: docs/core/01_ARCHITECTURE.md
   - Find database schema for {EXISTING_FEATURE}
   - Report: Tables and relationships

3. Check code:
   - /src/types/shared/ - Find feature types
   - /src/services/ - Find feature service
   - /src/repositories/ - Find feature repository
   - Report: Current code structure

4. Read: docs/core/00_FOUNDATION_ARCHITECTURE.md
   - Which layer(s) need changes?
   - Report: Layers to modify

PHASE 2: DISCOVERY
5. Search: "Is {NEW_CAPABILITY} already there?"
   - Check existing types for similar concepts
   - Check existing service methods
   - Report: Whether this is truly new or existing

6. Identify what needs to change:
   - Type definitions? YES/NO
   - Database schema? YES/NO
   - Service logic? YES/NO
   - API endpoints? YES/NO
   - Report: Change list

PHASE 3: IMPLEMENTATION
- Extend types if needed
- Extend service with new method
- Extend repository if DB change needed
- Add/update routes
- Add tests

PHASE 4: UPDATE EXISTING DOCS
- docs/core/02_FEATURES.md - Update {EXISTING_FEATURE} section
- docs/core/01_ARCHITECTURE.md - Update schema if changed
- docs/core/ERROR_CODES.md - Add new error codes if any

NO new .md files for this extension.
```

---

## 🎯 Template for ANY Feature

**If none of the above fit your use case, use this:**

```
Add {FEATURE_NAME} to SmartKhata

BEFORE CODING, READ THESE DOCS:
1. docs/core/02_FEATURES.md - Does this feature exist?
2. docs/core/01_ARCHITECTURE.md - Where in the database?
3. docs/core/00_FOUNDATION_ARCHITECTURE.md - Which layer owns this?
4. /src/types/shared/ - What types can I reuse?
5. docs/core/ERROR_CODES.md - What errors might happen?
6. docs/core/ARCHITECTURE_DECISION_MATRIX.md - Any design decisions needed?

REPORT YOUR FINDINGS:
- Feature exists? YES/NO
- Database changes needed? YES/NO
- Service/Repository changes? YES/NO
- New types needed? YES/NO
- New error codes? YES/NO
- Architecture decisions? [list]

AFTER I APPROVE:
- Implement code (no .md files)
- Update existing docs only:
  - docs/core/01_ARCHITECTURE.md
  - docs/core/02_FEATURES.md
  - docs/core/ERROR_CODES.md
  - docs/core/ENTITY_RELATIONSHIPS.md

Never create new .md files for this feature.
```

---

## 🚨 Red Flags: When to Stop Copilot

If Copilot is doing ANY of these, **STOP and redirect it**:

```
❌ Creating new .md files (e.g., DISCOUNT_SPEC.md)
   → Say: "Don't create new docs. Update docs/core/02_FEATURES.md instead"

❌ Skipping the discovery phase
   → Say: "First read docs/core/02_FEATURES.md and 01_ARCHITECTURE.md"

❌ Modifying database without checking schema
   → Say: "Read docs/core/01_ARCHITECTURE.md first"

❌ Not adding types to /src/types/shared/
   → Say: "Add types to /src/types/shared/ following existing patterns"

❌ Ignoring existing services/repositories
   → Say: "Check existing services first, extend don't duplicate"

❌ Creating feature-specific error codes without central registry
   → Say: "Add to docs/core/ERROR_CODES.md"

❌ Hard-deleting data instead of soft delete
   → Say: "Use soft delete (is_deleted flag) per 00_FOUNDATION_ARCHITECTURE.md"

❌ Not updating 02_FEATURES.md
   → Say: "Update docs/core/02_FEATURES.md with this feature"
```

---

## 💡 How This Works

**Example: You ask Copilot to add discount feature**

```
YOU: [Copy "Simple Feature" prompt above and paste into Copilot]

COPILOT does discovery:
├─ Reads 02_FEATURES.md → "No discount feature"
├─ Reads 01_ARCHITECTURE.md → "Add discounts table"
├─ Reads 00_FOUNDATION_ARCHITECTURE.md → "InvoiceService owns this"
├─ Checks /src/types/shared/ → "Reuse Party type, add DiscountRequest"
└─ Checks ERROR_CODES.md → "Need DISCOUNT_NOT_FOUND error"

COPILOT reports:
"Feature exists? NO
Database table: discounts (party_id, discount_percent, min_quantity)
Service: InvoiceService
Reusable types: Party, Invoice
New error codes: DISCOUNT_NOT_FOUND, DISCOUNT_INACTIVE"

YOU: "Yes, proceed"

COPILOT implements:
├─ /src/types/shared/discount.types.ts
├─ /src/services/InvoiceService.ts (add discount methods)
├─ /src/repositories/DiscountRepository.ts
├─ /src/routes/v2/discounts.ts
└─ /src/__tests__/discounts.test.ts

COPILOT updates docs:
├─ 01_ARCHITECTURE.md → adds discounts schema
├─ 02_FEATURES.md → adds discount feature
├─ ERROR_CODES.md → adds new error codes
└─ ENTITY_RELATIONSHIPS.md → adds discount links

RESULT: ✅ Feature added, docs updated, no duplication
```

---

## 📝 Quick Reference

| Scenario | Use | Find It |
|----------|-----|---------|
| Simple single-table feature | [Simple Feature](#simple-feature) | Search "Simple Feature" |
| Feature touching multiple parts | [Complex Feature](#complex-feature) | Search "Complex Feature" |
| External API integration | [Integration Feature](#integration-feature) | Search "Integration" |
| Bug fix | [Bug Fix](#bug-fix) | Search "Bug Fix" |
| Multi-platform (iOS/Android/Web/Windows) | [Multi-Platform](#multi-platform-feature-ios-android-web-windows) | Search "Multi-Platform" |
| Extend existing feature | [Extending](#extending-existing-feature) | Search "Extending" |
| Anything else | [Template](#template-for-any-feature) | Search "Template" |

---

## ✅ Success Criteria

When Copilot finishes, verify:

- [ ] Feature code created (no .md files created)
- [ ] Existing 4 docs updated (02_FEATURES, 01_ARCHITECTURE, ERROR_CODES, ENTITY_RELATIONSHIPS)
- [ ] Types added to /src/types/shared/
- [ ] Service/Repository created or extended
- [ ] Tests written
- [ ] No duplicate documentation
- [ ] Follows naming conventions (snake_case DB, camelCase API)
- [ ] Error handling included
- [ ] Documentation links are accurate

If any of the above is missing, ask Copilot to fix before merging.

---

## 🔗 Related Files

- [docs/core/00_FOUNDATION_ARCHITECTURE.md](00_FOUNDATION_ARCHITECTURE.md) - Architecture overview
- [docs/core/02_FEATURES.md](02_FEATURES.md) - Feature registry
- [docs/core/01_ARCHITECTURE.md](01_ARCHITECTURE.md) - Database schema
- [docs/core/HOW_TO_ADD_FEATURES.md](HOW_TO_ADD_FEATURES.md) - Detailed workflow
- [docs/core/ARCHITECTURE_DECISION_MATRIX.md](ARCHITECTURE_DECISION_MATRIX.md) - Design decisions
