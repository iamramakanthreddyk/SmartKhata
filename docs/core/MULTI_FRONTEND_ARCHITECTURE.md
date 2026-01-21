# Multi-Frontend Architecture: iOS, Android, Web, Windows

**Purpose**: Show how shared types enable ANY frontend to work with backend  
**Audience**: Frontend developers, architects  
**Status**: REFERENCE

---

## 🎯 The Challenge

**Your backend serves**:
- React Native mobile app (iOS/Android)
- React web app (Web)
- Flutter app (Android/iOS)
- Windows desktop app (Electron/WinUI)
- Any other frontend (Vue, Svelte, etc.)

**All using the SAME backend**, WITHOUT changes.

---

## ✅ Solution: Shared Types + Versioned APIs

### Architecture Overview

```
┌─────────────────────────────────────┐
│ iOS App (React Native/Swift)        │
├─────────────────────────────────────┤
│ Android App (React Native/Kotlin)   │
├─────────────────────────────────────┤
│ Web App (React/Vue/Svelte)          │
├─────────────────────────────────────┤
│ Windows App (Electron/WinUI)        │
└────────────┬────────────────────────┘
             │ HTTP REST API (JSON)
             │ Shared Types (TypeScript)
             ↓
┌─────────────────────────────────────┐
│ Backend (Node.js)                   │
├─────────────────────────────────────┤
│ Shared Types: /src/types/shared/    │
│ API Contract: Versioned (/v1, /v2)  │
│ Database: PostgreSQL                │
└─────────────────────────────────────┘
```

---

## 🔄 How Shared Types Work

### Step 1: Define Types Once in Backend

**File**: `backend/src/types/shared/invoice.types.ts`

```typescript
/**
 * Shared types used by ALL frontends
 * ✅ iOS app imports these
 * ✅ Android app imports these
 * ✅ Web app imports these
 * ✅ Windows app imports these
 */

// DATABASE FORMAT (snake_case)
export interface InvoiceDbDto {
  id: string;
  business_id: string;
  invoice_number: string;
  party_id: string;
  subtotal: number;
  gst_amount: number;
  total_amount: number;
  status: 'draft' | 'issued' | 'paid' | 'cancelled';
  created_at: string;
  updated_at: string;
}

// API FORMAT (camelCase - what all frontends receive)
export interface InvoiceApiDto {
  id: string;
  businessId: string;
  invoiceNumber: string;
  partyId: string;
  subtotal: number;
  gstAmount: number;
  totalAmount: number;
  status: 'draft' | 'issued' | 'paid' | 'cancelled';
  createdAt: string;
  updatedAt: string;
}

// REQUEST FORMAT (what all frontends send)
export interface CreateInvoiceRequest {
  partyId: string;
  invoiceDate: string;
  items: InvoiceItemRequest[];
}

export interface InvoiceItemRequest {
  inventoryId: string;
  quantity: number;
  unitPrice: number;
  gstRate: number;
}
```

### Step 2: Frontend Imports Same Types

**React Native iOS App** (`mobile-app/ios/src/types/index.ts`)
```typescript
// iOS app imports same types from shared
export type { InvoiceApiDto, CreateInvoiceRequest } from '@shared/types';
```

**React Web App** (`web-app/src/types/index.ts`)
```typescript
// Web app imports same types from shared
export type { InvoiceApiDto, CreateInvoiceRequest } from '@shared/types';
```

**Flutter Android App** (converted to Dart)
```dart
// Android would auto-generate or manually convert
// Same interface contract, different language
class InvoiceApiDto {
  final String id;
  final String businessId;
  final String invoiceNumber;
  // ... same fields
}
```

### Step 3: All Frontends Use Same Types

**iOS App Component** (React Native)
```typescript
import type { InvoiceApiDto } from '@types';

export function InvoiceScreen() {
  const [invoice, setInvoice] = useState<InvoiceApiDto | null>(null);

  const loadInvoice = async (id: string) => {
    const response = await fetch(`/api/v2/invoices/${id}`);
    const data: InvoiceApiDto = await response.json();  // ← Type-safe!
    setInvoice(data);
  };

  return (
    <View>
      <Text>{invoice?.invoiceNumber}</Text>  {/* ✅ Type-checked */}
      <Text>{invoice?.totalAmount}</Text>
    </View>
  );
}
```

**Web App Component** (React)
```typescript
import type { InvoiceApiDto } from '@types';

export function InvoiceCard() {
  const [invoice, setInvoice] = useState<InvoiceApiDto | null>(null);

  const loadInvoice = async (id: string) => {
    const response = await fetch(`/api/v2/invoices/${id}`);
    const data: InvoiceApiDto = await response.json();  // ← Same type!
    setInvoice(data);
  };

  return (
    <div>
      <p>{invoice?.invoiceNumber}</p>  {/* ✅ Type-checked */}
      <p>{invoice?.totalAmount}</p>
    </div>
  );
}
```

---

## 🌐 How This Enables Multi-Frontend Support

### Without Shared Types (WRONG)

```
iOS App defines:
interface Invoice {
  id: string;
  invoiceNumber: string;  // camelCase
}

Android App defines:
interface Invoice {
  id: string;
  invoice_number: string; // snake_case - WRONG!
}

Web App defines:
interface Invoice {
  id: string;
  invoice_no: string;     // Different name - WRONG!
}

Backend returns:
{
  "invoiceNumber": "INV-001"
}

Result:
├─ iOS app ✅ works (matches invoiceNumber)
├─ Android app ❌ crashes (expects invoice_number)
└─ Web app ❌ crashes (expects invoice_no)
```

### With Shared Types (CORRECT)

```
/src/types/shared/invoice.types.ts defines:
interface InvoiceApiDto {
  id: string;
  invoiceNumber: string;  // ← SINGLE SOURCE OF TRUTH
}

iOS app:
import type { InvoiceApiDto } from '@types';
// ✅ Uses invoiceNumber

Android app:
import type { InvoiceApiDto } from '@types';
// ✅ Uses invoiceNumber

Web app:
import type { InvoiceApiDto } from '@types';
// ✅ Uses invoiceNumber

Backend returns:
{
  "invoiceNumber": "INV-001"
}

Result:
├─ iOS app ✅ works
├─ Android app ✅ works
└─ Web app ✅ works
```

---

## 📦 Shared Types Package Structure

### Option 1: Monorepo (Recommended)

```
SmartKhata/
├── backend/
│   ├── src/
│   │   ├── types/shared/           ← Types here
│   │   │   ├── invoice.types.ts
│   │   │   ├── party.types.ts
│   │   │   └── index.ts
│   │   ├── services/
│   │   └── routes/
│   └── package.json
│
├── frontend-web/
│   ├── src/
│   │   ├── types/
│   │   │   └── index.ts            ← Imports from backend
│   │   └── components/
│   └── package.json
│
├── frontend-mobile/
│   ├── src/
│   │   ├── types/
│   │   │   └── index.ts            ← Imports from backend
│   │   └── screens/
│   └── package.json
│
└── shared-types/
    ├── src/types/
    │   ├── invoice.types.ts
    │   ├── party.types.ts
    │   └── index.ts
    └── package.json
```

**tsconfig.json** (all projects)
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@types/*": ["../shared-types/src/types/*"],
      "@shared/*": ["../shared-types/src/*"]
    }
  }
}
```

### Option 2: NPM Package (Also Works)

```bash
# Publish shared types as npm package
npm publish @smartkhata/types

# Each frontend installs
npm install @smartkhata/types
```

**backend/package.json**
```json
{
  "dependencies": {
    "@smartkhata/types": "1.0.0"
  }
}
```

**frontend-web/package.json**
```json
{
  "dependencies": {
    "@smartkhata/types": "1.0.0"  // ← Same version!
  }
}
```

**frontend-mobile/package.json**
```json
{
  "dependencies": {
    "@smartkhata/types": "1.0.0"  // ← Same version!
  }
}
```

---

## 🔄 How APIs Stay in Sync

### API Versioning Ensures Compatibility

```
Backend API v1:
GET /api/v1/invoices/:id
Returns: {invoiceNumber, partyId, totalAmount}

Backend API v2 (with new discount feature):
GET /api/v2/invoices/:id
Returns: {invoiceNumber, partyId, totalAmount, discountAmount}

All frontends can:
├─ Old frontends continue using /api/v1 (v1 types still valid)
├─ New frontends use /api/v2 (v2 types with discountAmount)
└─ Mix and match as they upgrade
```

### Shared Types Stay in Sync

```
@smartkhata/types@1.0.0:
export interface InvoiceApiDto {
  id: string;
  invoiceNumber: string;
  partyId: string;
  totalAmount: number;
}

All frontends using @smartkhata/types@1.0.0:
├─ iOS app ✅ has all fields
├─ Android app ✅ has all fields
├─ Web app ✅ has all fields
└─ Windows app ✅ has all fields
```

---

## 📝 Example: Adding Discount Feature (Multi-Frontend)

### Step 1: Backend Adds Discount

**backend/src/types/shared/invoice.types.ts**
```typescript
export interface InvoiceApiDto {
  // ... existing fields
  totalAmount: number;
  discountAmount: number;  // ← NEW FIELD
  finalAmount: number;     // ← NEW FIELD
}
```

### Step 2: Update Shared Types Version

```bash
cd shared-types
npm version patch  # 1.0.0 → 1.0.1
npm publish
```

### Step 3: All Frontends Update

**iOS App**
```bash
npm update @smartkhata/types
```

**Web App**
```bash
npm update @smartkhata/types
```

**Android App**
```bash
npm update @smartkhata/types
```

**Windows App**
```bash
npm update @smartkhata/types
```

### Step 4: All Frontends Can Use New Fields

```typescript
// All frontends automatically get discountAmount
<Text>{invoice?.discountAmount}</Text>  // ✅ Now available everywhere!
```

---

## 🎯 Multi-Frontend Benefits

| Scenario | Benefit |
|----------|---------|
| Add new field to API | Update types once, all frontends get it |
| Fix field naming | Fix in backend types, all frontends fixed |
| Change field type | Update types, all frontends recompile |
| Add new feature | All frontends can use it immediately |
| Breaking change | Create v2 API, v1 types still work |

---

## 🚨 How to Break It (And How to Avoid)

### ❌ WRONG: Inconsistent Naming

```
Backend returns: { invoiceNumber: "INV-001" }
iOS app expects: { invoice_number }  // ❌
Android expects: { invoiceno }       // ❌
```

**Solution**: Use shared types, enforces same naming everywhere

### ❌ WRONG: Inconsistent Data Types

```
Backend type: { amount: number }
iOS overrides: { amount: string }  // ❌ Different type!
```

**Solution**: Import types from shared, can't override

### ❌ WRONG: Different API Versions

```
iOS app calls: /api/v1/invoices
Web app calls: /api/v2/invoices
Android calls: /api/v3/invoices
// All expecting different response shapes!
```

**Solution**: Specify version in shared types package, all upgrade together

### ✅ CORRECT: Single Source of Truth

```
backend/src/types/shared/invoice.types.ts
├─ iOS app imports
├─ Web app imports
├─ Android app imports
└─ Windows app imports
   └─ All use exact same types
```

---

## 📱 Practical Example: Running Feature Across All Platforms

### Scenario: Add "Payment Status" Feature

#### Step 1: Backend Adds Type

**backend/src/types/shared/invoice.types.ts**
```typescript
export type PaymentStatus = 'pending' | 'partial' | 'completed' | 'overdue';

export interface InvoiceApiDto {
  // ... existing fields
  paymentStatus: PaymentStatus;  // ← NEW
}
```

#### Step 2: Backend Implements Service & API

**backend/src/services/InvoiceService.ts**
```typescript
async getInvoice(id: string): Promise<InvoiceApiDto> {
  const invoice = await this.repo.findById(id);
  
  // Calculate payment status
  const paid = calculatePaidAmount(invoice.id);
  const status: PaymentStatus =
    paid === 0 ? 'pending' :
    paid < invoice.total_amount ? 'partial' :
    paid >= invoice.total_amount ? 'completed' :
    'overdue';
  
  return transformDbToApi({ ...invoice, payment_status: status });
}
```

**backend/src/routes/v2/invoices.ts**
```typescript
router.get('/invoices/:id', async (req, res) => {
  const invoice = await invoiceService.getInvoice(req.params.id);
  res.json(invoice);  // ← returns paymentStatus
});
```

#### Step 3: Publish Updated Types

```bash
cd shared-types
npm version minor  # 1.0.0 → 1.1.0 (new feature)
npm publish
```

#### Step 4: iOS App Uses Feature

**mobile-app/package.json**
```json
{
  "dependencies": {
    "@smartkhata/types": "^1.1.0"
  }
}
```

**mobile-app/src/screens/InvoiceScreen.tsx**
```typescript
import type { InvoiceApiDto, PaymentStatus } from '@smartkhata/types';

export function InvoiceScreen() {
  const [invoice, setInvoice] = useState<InvoiceApiDto | null>(null);

  const getStatusColor = (status: PaymentStatus) => {
    switch(status) {
      case 'pending': return 'red';
      case 'partial': return 'orange';
      case 'completed': return 'green';
      case 'overdue': return 'darkred';
    }
  };

  return (
    <View>
      <Text style={{ color: getStatusColor(invoice?.paymentStatus) }}>
        {invoice?.paymentStatus?.toUpperCase()}
      </Text>
    </View>
  );
}
```

#### Step 5: Web App Uses Feature (Same Code Pattern)

**web-app/package.json**
```json
{
  "dependencies": {
    "@smartkhata/types": "^1.1.0"
  }
}
```

**web-app/src/components/InvoiceCard.tsx**
```typescript
import type { InvoiceApiDto, PaymentStatus } from '@smartkhata/types';

export function InvoiceCard() {
  const [invoice, setInvoice] = useState<InvoiceApiDto | null>(null);

  const getStatusBadge = (status: PaymentStatus) => {
    const badgeClasses: Record<PaymentStatus, string> = {
      'pending': 'badge-danger',
      'partial': 'badge-warning',
      'completed': 'badge-success',
      'overdue': 'badge-dark-red',
    };
    return badgeClasses[status];
  };

  return (
    <div>
      <span className={getStatusBadge(invoice?.paymentStatus)}>
        {invoice?.paymentStatus}
      </span>
    </div>
  );
}
```

#### Step 6: Android App Uses Feature (Same Types)

**android-app/package.json**
```json
{
  "dependencies": {
    "@smartkhata/types": "^1.1.0"
  }
}
```

**android-app/src/screens/InvoiceScreen.tsx**
```typescript
import type { InvoiceApiDto, PaymentStatus } from '@smartkhata/types';

export function InvoiceScreen() {
  const [invoice, setInvoice] = useState<InvoiceApiDto | null>(null);

  const getStatusColor = (status: PaymentStatus) => {
    switch(status) {
      case 'pending': return '#FF0000';
      case 'partial': return '#FFA500';
      case 'completed': return '#00FF00';
      case 'overdue': return '#8B0000';
    }
  };

  return (
    <View>
      <Text style={{ color: getStatusColor(invoice?.paymentStatus) }}>
        {invoice?.paymentStatus}
      </Text>
    </View>
  );
}
```

---

## 🎯 Result

**One feature, all platforms:**
- ✅ iOS app shows payment status
- ✅ Android app shows payment status
- ✅ Web app shows payment status
- ✅ Windows app shows payment status
- ✅ All types aligned
- ✅ All use same API
- ✅ One source of truth

---

## 📋 Checklist: Multi-Frontend Setup

- [ ] Define shared types in `backend/src/types/shared/`
- [ ] Export types package (monorepo or npm)
- [ ] All frontends import from shared types
- [ ] tsconfig paths configured for each app
- [ ] API returns types that match shared definitions
- [ ] Adapters transform DB format → API format
- [ ] All frontends use same API version
- [ ] Tests verify types match between backend and frontend
- [ ] When adding feature: Update shared types first, then all frontends pick it up

---

**Result**: Any frontend (iOS, Android, Web, Windows, Flutter, etc.) can use the same backend without any modifications.** ✅
