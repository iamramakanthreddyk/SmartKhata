# 1. SmartKhata Architecture & Database Design

**Purpose**: System design philosophy, architecture layers, database schema, and technical foundation  
**Audience**: Backend developers, DevOps, system architects  
**Priority**: CRITICAL - Read first before implementation

---

## 🎯 Core Architecture Philosophy

### 5 Principles (Never Break These)
1. **Mobile First** - Design for mobile, enhance for web
2. **No Duplication** - Single source of truth for every concept
3. **Clear Separation** - Distinct layers: API, Business Logic, Data Access, UI
4. **Easy Maintenance** - Simple, clear, documented, no magic
5. **Easy Expansion** - Add features without touching existing code

### 6-Layer Architecture

```
┌─────────────────────────────────────────────────────┐
│  PRESENTATION LAYER (Mobile App)                    │
│  ├─ React Native UI Components                      │
│  ├─ Redux State Management                          │
│  └─ Local SQLite Cache                              │
└────────────┬────────────────────────────────────────┘
             │ HTTP REST API (JSON)
             ↓
┌─────────────────────────────────────────────────────┐
│  API LAYER (Node.js REST Server)                    │
│  ├─ Request Validation (Zod)                        │
│  ├─ Request Router                                  │
│  └─ Response Formatting                             │
└────────────┬────────────────────────────────────────┘
             │
┌─────────────────────────────────────────────────────┐
│  BUSINESS LOGIC LAYER (Services)                    │
│  ├─ Party Service                                   │
│  ├─ Inventory Service                               │
│  ├─ Invoice Service                                 │
│  ├─ Ledger Service                                  │
│  └─ ... (One per domain)                            │
└────────────┬────────────────────────────────────────┘
             │
┌─────────────────────────────────────────────────────┐
│  DATA ACCESS LAYER (Repositories)                   │
│  ├─ PartyRepository (Query & Cache)                 │
│  ├─ InventoryRepository (Query & Cache)             │
│  └─ ... (One per entity)                            │
└────────────┬────────────────────────────────────────┘
             │
┌─────────────────────────────────────────────────────┐
│  DATABASE LAYER (PostgreSQL with RLS)               │
│  ├─ Tables (Normalized, ACID, Multi-tenant)         │
│  ├─ Row-Level Security (RLS)                        │
│  ├─ Indexes (Performance)                           │
│  ├─ Triggers (Business Rules)                       │
│  └─ Migrations (Version Control)                    │
└─────────────────────────────────────────────────────┘

EXTERNAL INTEGRATIONS (Side Effects):
├─ GST Portal (GSTR-1, E-way)
├─ Razorpay (Payments)
├─ SMS/WhatsApp (MSG91)
├─ Email (SendGrid)
└─ OCR (AWS Textract)
```

---

## 📱 Why Mobile-First?

- **Field usage**: Owner/staff uses app at counter/shop floor
- **Offline requirement**: No internet? App still works
- **Performance**: Mobile forces optimization
- **Simplicity**: Mobile UI is smaller = simpler features
- **Web is secondary**: Desktop is admin/reporting, not primary

---

## 🗄️ Database Schema (Normalized, Multi-Tenant with RLS)

### Core Tables

**1. businesses** - Each business/account
```sql
CREATE TABLE businesses (
  id UUID PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  gst_number VARCHAR(15),
  owner_email VARCHAR(255) UNIQUE,
  phone VARCHAR(20),
  address TEXT,
  city VARCHAR(100),
  state VARCHAR(50),
  country VARCHAR(50) DEFAULT 'India',
  currency VARCHAR(3) DEFAULT 'INR',
  subscription_plan VARCHAR(50) DEFAULT 'free',
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT unique_gst_number UNIQUE(gst_number)
);
-- Enable RLS: Users can only see their own business
ALTER TABLE businesses ENABLE ROW LEVEL SECURITY;
```

**2. staff** - Users/staff in a business
```sql
CREATE TABLE staff (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  email VARCHAR(255) UNIQUE NOT NULL,
  name VARCHAR(255) NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  phone VARCHAR(20),
  role VARCHAR(50), -- owner, manager, staff
  permissions JSONB DEFAULT '{}',
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
-- RLS: Staff can only see staff in their business
ALTER TABLE staff ENABLE ROW LEVEL SECURITY;
```

**3. parties** - Customers/vendors
```sql
CREATE TABLE parties (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  name VARCHAR(255) NOT NULL,
  type VARCHAR(50), -- customer, vendor, both
  email VARCHAR(255),
  phone VARCHAR(20),
  gst_number VARCHAR(15),
  address TEXT,
  opening_balance DECIMAL(12,2) DEFAULT 0,
  is_active BOOLEAN DEFAULT true,
  created_by UUID REFERENCES staff(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
ALTER TABLE parties ENABLE ROW LEVEL SECURITY;
```

**4. inventory** - Products/items
```sql
CREATE TABLE inventory (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  name VARCHAR(255) NOT NULL,
  sku VARCHAR(100),
  hsn_code VARCHAR(10),
  unit VARCHAR(50), -- kg, liter, piece, etc.
  quantity_on_hand DECIMAL(12,3) DEFAULT 0,
  purchase_price DECIMAL(12,2),
  selling_price DECIMAL(12,2),
  gst_rate DECIMAL(4,2) DEFAULT 18,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
ALTER TABLE inventory ENABLE ROW LEVEL SECURITY;
```

**5. invoices** - Sales invoices
```sql
CREATE TABLE invoices (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  invoice_number VARCHAR(50) NOT NULL,
  party_id UUID NOT NULL REFERENCES parties(id),
  invoice_date DATE NOT NULL,
  due_date DATE,
  subtotal DECIMAL(12,2),
  gst_amount DECIMAL(12,2),
  discount DECIMAL(12,2) DEFAULT 0,
  total_amount DECIMAL(12,2),
  status VARCHAR(50) DEFAULT 'draft', -- draft, issued, paid, cancelled
  irn VARCHAR(64), -- E-invoice reference
  payment_status VARCHAR(50) DEFAULT 'unpaid',
  notes TEXT,
  created_by UUID REFERENCES staff(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  CONSTRAINT unique_invoice UNIQUE(business_id, invoice_number)
);
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;
```

**6. invoice_items** - Line items in invoices
```sql
CREATE TABLE invoice_items (
  id UUID PRIMARY KEY,
  invoice_id UUID NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
  inventory_id UUID REFERENCES inventory(id),
  description VARCHAR(255),
  quantity DECIMAL(12,3),
  unit_price DECIMAL(12,2),
  gst_rate DECIMAL(4,2),
  line_total DECIMAL(12,2),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
ALTER TABLE invoice_items ENABLE ROW LEVEL SECURITY;
```

**7. ledger** - Party transaction history
```sql
CREATE TABLE ledger (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  party_id UUID NOT NULL REFERENCES parties(id),
  transaction_type VARCHAR(50), -- invoice, payment, adjustment
  reference_id UUID, -- invoice_id or payment_id
  debit DECIMAL(12,2) DEFAULT 0,
  credit DECIMAL(12,2) DEFAULT 0,
  balance DECIMAL(12,2),
  notes TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_party_balance (party_id, balance)
);
ALTER TABLE ledger ENABLE ROW LEVEL SECURITY;
```

**8. payments** - Payment receipts
```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  party_id UUID REFERENCES parties(id),
  payment_date DATE NOT NULL,
  amount DECIMAL(12,2),
  payment_method VARCHAR(50), -- cash, check, transfer, etc.
  reference_number VARCHAR(100),
  notes TEXT,
  created_by UUID REFERENCES staff(id),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
ALTER TABLE payments ENABLE ROW LEVEL SECURITY;
```

**9. audit_logs** - Security audit trail
```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  staff_id UUID REFERENCES staff(id),
  action VARCHAR(255),
  table_name VARCHAR(100),
  record_id UUID,
  old_values JSONB,
  new_values JSONB,
  ip_address VARCHAR(45),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_audit_search (table_name, record_id, created_at)
);
```

**10. business_settings** - Branding, defaults, customization
```sql
CREATE TABLE business_settings (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL UNIQUE REFERENCES businesses(id),
  
  -- Branding (Logo stored as S3 URL, not binary)
  logo_url TEXT,                    -- S3 path to logo image
  logo_uploaded_at TIMESTAMP,
  
  -- Colors (Hex format: #RRGGBB)
  primary_color VARCHAR(7) DEFAULT '#1976D2',
  secondary_color VARCHAR(7) DEFAULT '#DC004E',
  accent_color VARCHAR(7) DEFAULT '#4CAF50',
  
  -- Invoice customization
  invoice_prefix VARCHAR(20) DEFAULT 'INV',
  next_invoice_number INT DEFAULT 1,
  default_payment_terms VARCHAR(50) DEFAULT 'NET 30',
  invoice_footer TEXT,
  invoice_header TEXT,
  
  -- Defaults
  default_gst_rate INT DEFAULT 18,
  default_discount_percent DECIMAL(5,2) DEFAULT 0,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**11. invoice_templates** - Save & reuse invoice templates
```sql
CREATE TABLE invoice_templates (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  name VARCHAR(255) NOT NULL,       -- "Standard Invoice", "Advance Payment"
  description TEXT,
  
  -- Template data (pre-filled fields)
  default_party_id UUID REFERENCES parties(id),
  default_notes TEXT,
  
  -- Line items (stored as JSON array)
  template_items JSONB,            -- [{item_id, qty, rate, gst_rate}...]
  
  -- Flags
  is_default BOOLEAN DEFAULT false,
  usage_count INT DEFAULT 0,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, name)
);
```

**12. email_templates** - Save & reuse email messages
```sql
CREATE TABLE email_templates (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  name VARCHAR(255) NOT NULL,      -- "Invoice Reminder", "Payment Thank You"
  subject VARCHAR(255) NOT NULL,
  body TEXT NOT NULL,              -- HTML with {variables}
  category VARCHAR(50),            -- reminder, thankyou, urgent, invoice
  
  -- Variables allowed: {invoice_no}, {party_name}, {amount}, {due_date}
  is_default BOOLEAN DEFAULT false,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, name)
);
```

**13. sms_templates** - Save & reuse SMS messages
```sql
CREATE TABLE sms_templates (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  name VARCHAR(255) NOT NULL,      -- "Reminder", "Payment Alert"
  message TEXT NOT NULL,           -- Max 160 chars, supports {variables}
  category VARCHAR(50),            -- reminder, alert, payment, urgent
  
  character_count INT,             -- Auto-calculated for SMS length
  is_default BOOLEAN DEFAULT false,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, name)
);
```

**14. recurring_invoices** - Auto-billing setup
```sql
CREATE TABLE recurring_invoices (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  template_id UUID NOT NULL REFERENCES invoice_templates(id),
  
  party_id UUID NOT NULL REFERENCES parties(id),
  frequency VARCHAR(20) NOT NULL,  -- daily, weekly, monthly, quarterly, yearly
  
  start_date DATE NOT NULL,
  end_date DATE,                   -- NULL = ongoing forever
  next_invoice_date DATE NOT NULL,
  
  is_active BOOLEAN DEFAULT true,
  auto_finalize BOOLEAN DEFAULT false,  -- Auto-send to GST portal?
  auto_email BOOLEAN DEFAULT false,     -- Auto-email to customer?
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**15. payment_links** - Customer payment portal
```sql
CREATE TABLE payment_links (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  payment_link TEXT NOT NULL,      -- Unique URL: smartkhata.com/pay/xyz123
  qr_code_url TEXT,               -- QR code image (S3 path, optional custom upload)
  
  accepted_methods VARCHAR(100),  -- upi,card,netbanking,wallet
  
  payment_status VARCHAR(20) DEFAULT 'pending',  -- pending, partial, completed, expired
  amount_pending DECIMAL(12,2),   -- Original invoice amount
  amount_paid DECIMAL(12,2) DEFAULT 0,
  payment_received_at TIMESTAMP,
  
  expires_at TIMESTAMP,           -- Link expiration (optional)
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(invoice_id)
);
```

**16. invoice_reminders** - Track automated reminders
```sql
CREATE TABLE invoice_reminders (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  reminder_type VARCHAR(50),      -- pre_due, due, overdue_1day, overdue_5day, etc
  scheduled_date DATE,
  sent_date TIMESTAMP,
  
  channel VARCHAR(20),            -- email, sms, both
  recipient TEXT,                 -- email or phone
  
  status VARCHAR(20),             -- pending, sent, failed, bounced
  failure_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**17. payment_terms** - Payment term rules
```sql
CREATE TABLE payment_terms (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  name VARCHAR(100),              -- "NET 30", "NET 60", "2/10 NET 30"
  days_until_due INT,            -- 30 for NET 30, 60 for NET 60
  
  early_payment_discount DECIMAL(5,2),  -- 2% for paying within 10 days
  early_payment_days INT,        -- 10 for above example
  
  is_default BOOLEAN DEFAULT false,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**18. dashboard_metrics** - Analytics cache (refreshed nightly)
```sql
CREATE TABLE dashboard_metrics (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  metric_date DATE NOT NULL,      -- Today's date
  
  -- Revenue metrics
  total_invoiced DECIMAL(12,2),   -- All invoices for month
  total_received DECIMAL(12,2),   -- Paid invoices
  total_overdue DECIMAL(12,2),    -- Unpaid + overdue
  
  -- Collection metrics
  collection_rate DECIMAL(5,2),   -- % of invoices paid on time
  avg_payment_delay INT,          -- Days average delay
  
  -- Customer metrics
  top_customer_id UUID,           -- Highest revenue customer
  top_debtor_id UUID,             -- Most overdue customer
  
  -- Aging
  aging_0_30 DECIMAL(12,2),       -- Amount 0-30 days overdue
  aging_30_60 DECIMAL(12,2),
  aging_60_90 DECIMAL(12,2),
  aging_90_plus DECIMAL(12,2),
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**19. payment_methods** - Payment history & tracking
```sql
CREATE TABLE payment_methods (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  payment_method VARCHAR(50),     -- upi, card, netbanking, wallet, bank_transfer
  transaction_id TEXT,            -- Razorpay ID or bank reference
  
  amount_paid DECIMAL(12,2),
  payment_date TIMESTAMP NOT NULL,
  
  status VARCHAR(20),             -- success, failed, refunded
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**20. staff_roles** - Role-based access control
```sql
CREATE TABLE staff_roles (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  staff_id UUID NOT NULL REFERENCES staff(id),
  
  role VARCHAR(50) NOT NULL,      -- admin, manager, accountant, viewer
  
  -- Permissions (JSON for flexibility)
  permissions JSONB,              -- {create: true, edit: true, approve: true, delete: false}
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, staff_id)
);
```

**21. approval_workflows** - Invoice approval trail
```sql
CREATE TABLE approval_workflows (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  created_by_staff_id UUID NOT NULL REFERENCES staff(id),
  approved_by_staff_id UUID REFERENCES staff(id),
  
  status VARCHAR(20),             -- pending_approval, approved, rejected
  rejection_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  approved_at TIMESTAMP,
  
  INDEX idx_pending_approvals (status, created_at)
);
```

**22. customer_notifications** - Track customer engagement & notifications
```sql
CREATE TABLE customer_notifications (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  customer_id UUID NOT NULL REFERENCES parties(id),  -- Customer receiving notification
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  notification_type VARCHAR(50),   -- invoice_created, payment_reminder, receipt_sent
  channel VARCHAR(20),             -- sms, email, push
  recipient VARCHAR(255),          -- Phone number or email
  
  subject TEXT,
  message_body TEXT,
  
  status VARCHAR(20) DEFAULT 'pending', -- pending, sent, failed, opened (for email)
  sent_at TIMESTAMP,
  opened_at TIMESTAMP,
  failure_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_customer_recent (customer_id, created_at DESC),
  INDEX idx_status_pending (status, created_at)
);
```

**23. customer_network_profile** - Customer registration & engagement
```sql
CREATE TABLE customer_network_profile (
  id UUID PRIMARY KEY,
  customer_id UUID NOT NULL REFERENCES parties(id),  -- Links to existing customer
  
  -- Customer registration status
  phone_verified BOOLEAN DEFAULT false,
  phone_verified_at TIMESTAMP,
  email_verified BOOLEAN DEFAULT false,
  email_verified_at TIMESTAMP,
  
  -- Profile data
  profile_name VARCHAR(255),       -- Name they prefer (may differ from invoice name)
  profile_type VARCHAR(50),        -- individual, business, farmer, retailer, etc
  profile_photo_url TEXT,          -- S3 URL if provided
  
  -- Network engagement
  first_invoice_date DATE,         -- When first invoice created
  last_login_date TIMESTAMP,       -- Last portal access
  total_invoices_received INT DEFAULT 0,
  total_amount_received DECIMAL(12,2) DEFAULT 0,
  total_payments_made DECIMAL(12,2) DEFAULT 0,
  
  -- Engagement flags
  portal_access_enabled BOOLEAN DEFAULT true,
  notifications_enabled BOOLEAN DEFAULT true,
  preferred_language VARCHAR(10) DEFAULT 'hi',  -- For SMS/Email templates
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(customer_id)
);
```

**24. customer_engagement_metrics** - Analytics for network health
```sql
CREATE TABLE customer_engagement_metrics (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  metric_date DATE NOT NULL,
  
  -- Customer growth
  new_customers_count INT DEFAULT 0,     -- New customers added today
  verified_customers_count INT DEFAULT 0, -- Phone/email verified
  active_customers_count INT DEFAULT 0,   -- Login in last 7 days
  
  -- Engagement
  invoices_sent_count INT DEFAULT 0,      -- Invoices sent today
  notifications_sent_count INT DEFAULT 0, -- Notifications delivered
  notification_open_rate DECIMAL(5,2),    -- % of emails opened
  
  -- Payment influence
  avg_payment_days DECIMAL(5,2),          -- Days to payment (for customers notified)
  payment_velocity_impact DECIMAL(5,2),   -- % faster vs non-notified
  
  -- Network health
  portal_access_rate DECIMAL(5,2),        -- % of customers accessing portal
  repeat_customer_rate DECIMAL(5,2),      -- % with 2+ invoices
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, metric_date)
);
```

**25. invoice_bank_details** - Bank account info for invoice payments
```sql
CREATE TABLE invoice_bank_details (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  -- Account Information
  account_holder_name VARCHAR(255) NOT NULL,
  account_number VARCHAR(50) NOT NULL,
  ifsc_code VARCHAR(11) NOT NULL,
  bank_name VARCHAR(100),
  branch_name VARCHAR(100),
  
  -- Payment IDs for QR codes
  upi_id VARCHAR(100),                   -- "shop@upi"
  phone_pay_id VARCHAR(100),             -- "9876543210@phonepe"
  google_pay_id VARCHAR(100),
  
  -- Razorpay Account ID (for direct transfers)
  razorpay_account_id TEXT,
  
  is_default BOOLEAN DEFAULT true,
  is_active BOOLEAN DEFAULT true,
  
  upi_qr_url TEXT,                       -- S3 URL
  phone_pay_qr_url TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, account_number)
);
```

**26. po_master** - Purchase Order tracking
```sql
CREATE TABLE po_master (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  po_number VARCHAR(50) NOT NULL,        -- "PO-00001"
  po_date DATE NOT NULL,
  
  supplier_id UUID NOT NULL REFERENCES parties(id),
  po_status VARCHAR(50) DEFAULT 'draft', -- draft, issued, accepted, rejected, closed
  
  total_amount DECIMAL(12,2),
  tax_amount DECIMAL(12,2),
  grand_total DECIMAL(12,2),
  
  delivery_date DATE,
  payment_terms TEXT,
  notes TEXT,
  
  invoice_id UUID REFERENCES invoices(id),  -- Conversion to invoice
  
  created_by_staff_id UUID NOT NULL REFERENCES staff(id),
  accepted_by_supplier_id UUID REFERENCES staff(id),
  accepted_at TIMESTAMP,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, po_number)
);
```

**27. payment_qr_codes** - Multiple QR codes per invoice/PO
```sql
CREATE TABLE payment_qr_codes (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  qr_type VARCHAR(50),                   -- upi, phone_pay, google_pay, bank_transfer
  qr_code_url TEXT NOT NULL,
  qr_data TEXT,
  
  account_id UUID REFERENCES invoice_bank_details(id),
  amount DECIMAL(12,2),
  
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_invoice_type (invoice_id, qr_type)
);
```

---

## 🏗️ Codebase Structure (No Duplication)

```
SmartKhata/
│
├── BACKEND/
│   └── src/
│       ├── config/          (Environment, database config)
│       ├── middleware/      (Auth, permissions, error handling)
│       ├── routes/          (API endpoint definitions)
│       ├── controllers/     (Request handlers)
│       ├── services/        (Business logic)
│       │   ├── PartyService
│       │   ├── InventoryService
│       │   ├── InvoiceService
│       │   ├── PaymentService
│       │   ├── LedgerService
│       │   ├── AuthService
│       │   ├── GST Service
│       │   └── ...
│       ├── repositories/    (Data access - queries only)
│       ├── models/          (Sequelize/Prisma models)
│       ├── migrations/      (Database versions)
│       ├── validators/      (Input validation schemas)
│       ├── types/           (TypeScript interfaces)
│       ├── utils/           (Helpers - formatters, calculators)
│       ├── integrations/    (External API clients)
│       │   ├── GST Portal
│       │   ├── Razorpay
│       │   ├── MSG91
│       │   ├── SendGrid
│       │   └── AWS Textract
│       └── index.ts         (Entry point)
│
├── MOBILE/
│   └── src/
│       ├── screens/         (Page components)
│       │   ├── HomeScreen
│       │   ├── InvoiceScreen
│       │   ├── PartyScreen
│       │   ├── InventoryScreen
│       │   └── ...
│       ├── components/      (Reusable UI components)
│       ├── redux/           (State management)
│       │   ├── slices/      (Auth, Invoices, Parties, etc.)
│       │   └── selectors/
│       ├── api/             (API client - same URLs as backend)
│       ├── local-storage/   (Offline cache)
│       ├── utils/           (Shared utilities)
│       ├── types/           (Shared TypeScript types)
│       └── app.tsx          (Entry point)
│
└── SHARED/
    └── src/
        ├── types.ts         (Shared TypeScript interfaces)
        ├── constants.ts     (Shared constants - roles, statuses)
        ├── formatters.ts    (Currency, date, GST formatters)
        ├── calculators.ts   (GST, totals, balances)
        ├── validators.ts    (Reusable validation functions)
        └── utils.ts         (Common helpers)
```

**Golden Rule**: If code is used in 2+ places, move it to SHARED/

---

## 🔐 Row-Level Security (RLS) Strategy

Every query is automatically scoped to the user's business:

```sql
-- Create security context functions
CREATE OR REPLACE FUNCTION current_user_id() RETURNS UUID AS $$
  SELECT current_setting('app.current_user_id')::UUID;
$$ LANGUAGE SQL;

CREATE OR REPLACE FUNCTION current_business_id() RETURNS UUID AS $$
  SELECT current_setting('app.current_business_id')::UUID;
$$ LANGUAGE SQL;

-- RLS Policy: Users can only see invoices from their business
CREATE POLICY business_isolation_invoices ON invoices
  USING (business_id = current_business_id());

-- This prevents any user from accidentally (or intentionally) 
-- accessing another business's data
```

---

## 📊 Core Entities & Relationships

```
businesses (1) ──► (M) staff
                ├──► (M) parties
                ├──► (M) inventory
                └──► (M) invoices

parties (1) ──► (M) invoices
            ├──► (M) ledger
            └──► (M) payments

invoices (1) ──► (M) invoice_items
             ├──► (M) payments
             └──► (M) ledger

staff (1) ──► (M) audit_logs
```

---

## 📋 Implementation Checklist

- [ ] Create PostgreSQL database
- [ ] Run migrations to create tables
- [ ] Enable RLS on all tables
- [ ] Create security context functions
- [ ] Setup authentication middleware
- [ ] Create repositories for data access
- [ ] Setup TypeScript types
- [ ] Configure environment variables
- [ ] Test RLS isolation between businesses

---

**Related Documents**: 02_FEATURES.md, 03_API.md, 04_SECURITY.md, 07_DEVELOPMENT.md
