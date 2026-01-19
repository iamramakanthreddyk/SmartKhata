# 7. SmartKhata - Development Guide

**Purpose**: Code standards, setup, testing, Git workflow, CI/CD  
**Audience**: Backend developers, mobile developers, QA  
**Consolidates**: DEVELOPMENT_RULES.md, SETUP_GUIDE.md, TESTING_STRATEGY.md

---

## 💻 Development Rules (6 Core Rules)

### Rule 1: One Concept, One Place (No Duplication)

**If code used in 2+ places → Move to SHARED/**

**Example - RIGHT ✅**:
```typescript
// SHARED/src/calculators.ts
export function calculateGST(amount: number, rate: number): number {
  return (amount * rate) / 100;
}

// Backend/src/services/InvoiceService.ts
import { calculateGST } from 'shared';
const gst = calculateGST(subtotal, taxRate);

// Mobile/src/screens/InvoiceScreen.tsx
import { calculateGST } from 'shared';
const gst = calculateGST(subtotal, taxRate);
```

**Example - WRONG ❌**:
```typescript
// Backend duplicates logic
calculateGST(amount, rate) { return amount * rate / 100; }

// Mobile duplicates logic
const gst = subtotal * taxRate / 100;
// Leads to bugs when formula changes
```

---

### Rule 2: Keep Functions Small (< 30 lines)

**If function > 30 lines → Break it down**

```typescript
// WRONG ❌ - 50 lines
async function createInvoice(data) {
  // Validate input
  if (!data.party_id) throw new Error(...);
  if (!data.items) throw new Error(...);
  
  // Fetch party
  const party = await Party.findById(data.party_id);
  if (!party) throw new Error(...);
  
  // Calculate totals (5 lines)
  // Generate invoice number (3 lines)
  // Create in database (3 lines)
  // Update party ledger (3 lines)
  // Send email (3 lines)
  // Etc...
}

// RIGHT ✅ - Small, focused functions
async function createInvoice(data) {
  validateInvoiceInput(data);
  const party = await partyRepository.findById(data.party_id);
  
  const invoice = invoiceFactory.create(data, party);
  await invoiceRepository.save(invoice);
  await ledgerService.recordInvoice(invoice);
  await notificationService.sendInvoiceEmail(invoice);
  
  return invoice;
}
```

---

### Rule 3: Clear Names (No Abbreviations)

```typescript
// WRONG ❌
function calcTot(items, tx) { ... }
let inv, pts, amt;

// RIGHT ✅
function calculateTotal(items, taxRate) { ... }
let invoice, parties, amount;
```

---

### Rule 4: Handle Errors Explicitly

```typescript
// WRONG ❌ - Silent failure
try {
  const result = await fetchData();
} catch (e) {
  // ignoring error
}

// RIGHT ✅ - Explicit handling
try {
  const result = await fetchData();
} catch (error) {
  if (error instanceof NetworkError) {
    throw new APIError('Unable to reach server', 500);
  }
  logger.error('Unexpected error', error);
  throw new InternalServerError();
}
```

---

### Rule 5: Every Function Has Types (TypeScript)

```typescript
// WRONG ❌
function calculateGST(amount, rate) {
  return amount * rate / 100;
}

// RIGHT ✅
function calculateGST(amount: number, rate: number): number {
  return (amount * rate) / 100;
}

// RIGHT ✅ with complex types
function createInvoice(data: CreateInvoiceRequest): Promise<Invoice> {
  // ...
}
```

---

### Rule 6: Write Tests (Min 70% Coverage)

```typescript
// For every function, write tests

// calculateGST.test.ts
describe('calculateGST', () => {
  it('should calculate 18% GST correctly', () => {
    expect(calculateGST(100, 18)).toBe(18);
  });
  
  it('should handle zero amount', () => {
    expect(calculateGST(0, 18)).toBe(0);
  });
});
```

---

## 🚀 Development Workflow

### 1. Setup Local Environment

**Prerequisites**:
```
Node.js 18+
PostgreSQL 14+
React Native CLI
Git
```

**Initial Setup** (10 minutes):
```bash
# Clone repository
git clone https://github.com/smartkhata/smartkhata.git
cd smartkhata

# Install dependencies
npm install

# Setup database
createdb smartkhata_dev
npm run migrate:dev

# Start services
npm run dev
# API: http://localhost:3001
# Mobile: Localhost tunnel active
```

### 2. Create Feature Branch

```bash
# Create from main branch
git checkout main
git pull origin main

# Create feature branch (kebab-case)
git checkout -b feature/invoicing-system
# or
git checkout -b fix/payment-validation
# or
git checkout -b docs/api-guide
```

### 3. Develop & Test

```bash
# Run tests while developing
npm run test:watch

# Run linter to catch issues
npm run lint

# Run type checking
npm run type-check

# Build locally
npm run build
```

### 4. Commit Changes

```bash
# Use conventional commit format
git commit -m "feat: add invoice finalization flow"
git commit -m "fix: handle GST rate edge case"
git commit -m "docs: update API endpoint docs"

# Commits should be atomic (one change = one commit)
# NOT: "implemented invoicing, fixed bugs, updated docs"
```

### 5. Push & Create PR

```bash
# Push feature branch
git push origin feature/invoicing-system

# Create Pull Request on GitHub
# Include description of changes
# Reference any issues: Fixes #123
```

### 6. Code Review

- Minimum 2 approvals required
- All tests must pass
- No merge conflicts
- No regressions in performance

### 7. Merge & Deploy

```bash
# Merge with "Squash and merge" strategy
# Keeps main branch clean

# Auto-deployment triggers after merge
# → Staging environment (5 min)
# → Production (with monitoring)
```

---

## 🧪 Testing Strategy

### Test Pyramid

```
       ▲
      / \         E2E Tests (10% of tests)
     /   \        └─ Full user workflows
    / ─ ─ \
   /       \      Integration Tests (30% of tests)
  / ─ ─ ─ ─ \    └─ Services talking to database
 /           \
/ ─ ─ ─ ─ ─ ─ \ Unit Tests (60% of tests)
```

### Unit Tests (Backend)

```typescript
// Example: PartyService.ts test

describe('PartyService', () => {
  describe('createParty', () => {
    it('should create party with valid input', async () => {
      const result = await partyService.createParty({
        name: 'ABC Store',
        phone: '9876543210'
      });
      
      expect(result.id).toBeDefined();
      expect(result.name).toBe('ABC Store');
    });
    
    it('should throw error for invalid GST number', async () => {
      expect(() => partyService.createParty({
        name: 'ABC Store',
        gst_number: 'INVALID'
      })).toThrow('Invalid GST number');
    });
  });
});

// Run:  npm run test
// Watch:  npm run test:watch
// Coverage:  npm run test:coverage
```

### Integration Tests (API)

```typescript
describe('POST /api/invoices', () => {
  it('should create invoice and update ledger', async () => {
    const response = await request(app)
      .post('/api/invoices')
      .set('Authorization', `Bearer ${validToken}`)
      .send({
        party_id: 'uuid',
        items: [{...}]
      });
    
    expect(response.status).toBe(201);
    
    // Verify ledger was updated
    const ledger = await Ledger.findOne({
      invoice_id: response.body.id
    });
    expect(ledger).toBeDefined();
  });
});

// Run:  npm run test:integration
```

### E2E Tests (Mobile)

```typescript
// Mobile E2E test using Detox
describe('Invoice Creation Flow', () => {
  beforeAll(async () => {
    await device.launchApp();
  });
  
  it('should create invoice through 5-step flow', async () => {
    // Step 1: Select Party
    await waitFor(element(by.text('Select Customer')))
      .toBeVisible()
      .withTimeout(5000);
    await element(by.text('ABC Store')).tap();
    
    // Step 2: Add Items
    await element(by.id('add-item-btn')).tap();
    await typeText(by.id('quantity-input'), '5');
    
    // ... continue through all 5 steps
    
    // Verify invoice created
    await expect(element(by.text('Invoice created!'))).toBeVisible();
  });
});

// Run:  npm run test:e2e:mobile
```

### Performance Tests

```bash
# Load testing with 100 concurrent users
npm run test:performance

# Checks:
# - No endpoint takes >200ms (p99)
# - Error rate stays <0.1%
# - Memory doesn't leak
```

---

## ✅ Quality Gates (CI/CD)

**Before merge**:
- ✅ All unit tests pass
- ✅ Code coverage >70%
- ✅ No lint errors
- ✅ No TypeScript errors
- ✅ No security vulnerabilities (npm audit)
- ✅ Code review approved (2 reviewers)

**After merge**:
- ✅ Build succeeds
- ✅ Staging deployment succeeds
- ✅ Smoke tests pass on staging
- ✅ Performance benchmarks met

---

## 📝 Naming Conventions

**Files**:
- Components: `PartyList.tsx`
- Services: `PartyService.ts`
- Tests: `PartyService.test.ts`
- Utilities: `partyCalculators.ts`

**Functions**:
```typescript
// Prefix with action
getParties()       // Fetch from API
createParty()      // Create new
updateParty()      // Modify existing
deleteParty()      // Remove
calculateGST()     // Compute
validateGST()      // Check validity
parseCSV()         // Parse input
formatCurrency()   // Format output
```

**Variables**:
```typescript
// Use descriptive names
const totalAmount = 1000;
const gstRate = 18;
const invoiceList = [];

// NOT: tot, gst, list
```

---

**Related Documents**: 01_ARCHITECTURE.md, 03_API.md, 04_SECURITY.md, 08_UX.md
