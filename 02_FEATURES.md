# 2. SmartKhata - Complete Feature Specifications

**Purpose**: All 35 features across 6 categories with technical specs  
**Audience**: Product managers, developers, QA  
**Status**: ✅ Consolidated from all FEATURE_*.md files

This document replaces: FEATURE_INVOICING.md, FEATURE_EINVOICING.md, FEATURE_PURCHASE_ENTRY.md, FEATURE_EXPENSE_TRACKING.md, FEATURE_RECONCILIATION.md

---

## 📋 Feature Categories (49 Total - Increased from 48)

### 1. Core Features (5)
1. **Invoicing** - Create and manage sales invoices with 5-step flow
2. **Parties** - Customer/vendor database with outstanding tracking  
3. **Inventory** - Product catalog with pricing and stock
4. **Ledger** - Real-time party transaction history
5. **Payments** - Record payments received/made

### 2. Compliance (3)
6. **E-Invoicing** - GST portal integration with IRN generation
7. **E-Way Bills** - Goods movement documentation
8. **GST Reports** - GSTR-1, GSTR-2, reconciliation

### 3. Accounting (3)
9. **Purchase Entry** - Record purchase invoices (manual + OCR + AI)
10. **Expenses** - Track business expenses with categorization
11. **Reconciliation** - AI-powered payment matching

### 4. Sharing & Distribution (5)
12. **Email** - Send invoices via email with tracking
13. **WhatsApp** - Share invoices via WhatsApp
14. **SMS** - Payment reminders and notifications
15. **PDF** - Custom PDF generation with branding
16. **Print** - Normal and thermal printer support

### 5. Advanced (18) - EXPANDED
17. **Reports** - P&L, Cash Flow, GST analysis
18. **Shared Ledger** - Customer portal (read-only access)
19. **POS** - Point of sale mode for retail
20. **Online Store** - Customer self-service invoicing
21. **Invoice Templates** - Save & reuse templates (NEW)
22. **Recurring Invoices** - Auto-billing on schedule (NEW)
23. **Email Templates** - Reusable email messages (NEW)
24. **SMS Templates** - Reusable SMS messages (NEW)
25. **Payment Portal** - Customer online payment with QR code (CRITICAL)
26. **Auto-Reminders** - Automated payment follow-ups (CRITICAL)
27. **Overdue Dashboard** - Real-time overdue tracking (CRITICAL)
28. **Analytics Dashboard** - Revenue, cash flow, customer insights (CRITICAL)
29. **Payment Terms** - Discount rules & flexible payment options
30. **Multi-Staff Management** - Role-based access with approvals
31. **Payment Tracking** - Track payment status & receipts
32. **Customer Portal** - Full self-service access
33. **Customer Discovery & Network** - Search, auto-register, notifications (CRITICAL)
34. **Purchase Orders & Advanced Invoicing** - PO generation, bank details, multi-QR (NEW)

### 6. Integration (4)
33. **Tally API** - Tally integration for data sync
34. **REST API** - Custom integrations
35. **Custom Themes** - White-label styling with branding
28. **Bulk Operations** - Import/export data

---

## ✅ Core Features Details

### Feature 1: Invoicing

**What it does**: Create, manage, and track sales invoices

**5-Step Guided Flow**:
1. Select Party → 2. Add Items → 3. Add Charges → 4. Customize → 5. Generate

**Database Impact**:
- Creates entry in `invoices` table
- Creates entries in `invoice_items` table  
- Triggers automatic ledger entry (credit)
- Updates party outstanding balance

**API Endpoints**:
```
POST   /api/invoices                 - Create invoice
GET    /api/invoices                 - List invoices
GET    /api/invoices/:id             - Get details
PUT    /api/invoices/:id             - Update (draft only)
DELETE /api/invoices/:id             - Delete draft
POST   /api/invoices/:id/finalize    - Lock invoice
GET    /api/invoices/:id/pdf         - Download PDF
```

**Mobile UI**: See 08_UX.md for detailed screen flows

**Availability by Plan**:
- FREE: 15 invoices/month
- DIAMOND: 50/month
- PLATINUM: 200/month
- ENTERPRISE: Unlimited
- CUSTOM: Unlimited

**Cannot Disable**: Core feature always enabled

**Auto-Calculations**:
- Line totals = qty × unit price
- Subtotal = sum of line totals
- Tax calculation (depends on party location):
  - **Intra-state** (same state as business): GST splits to CGST (50%) + SGST (50%)
    - Example: 18% rate = 9% CGST + 9% SGST
  - **Inter-state** (different state): Single IGST rate
    - Example: 18% rate = 18% IGST
- Tax amount = Subtotal × tax rate
- Final total = Subtotal + Tax - Discount
- Reverse charge: If party is unregistered/composition, handling depends on rules

---

### Feature 2: Parties

**What it does**: Maintain customer and supplier database

**Database Impact**:
- Creates entries in `parties` table
- Links to `invoices` (customer) and `payments` (both)
- Tracks opening balance in `ledger`

**API Endpoints**:
```
POST   /api/parties                  - Create party
GET    /api/parties                  - List parties
GET    /api/parties/:id              - Get details
PUT    /api/parties/:id              - Update party
DELETE /api/parties/:id              - Soft delete
GET    /api/parties/:id/ledger       - Get transaction history
GET    /api/parties/:id/outstanding  - Get outstanding amount
```

**Key Fields**:
- Name, Email, Phone
- GST Number (for tax compliance)
- Address (for shipping)
- Opening Balance (for old customers)
- Party Type (customer/vendor/both)
- Active status

---

### Feature 3: Inventory

**What it does**: Manage product catalog with pricing and stock

**Database Impact**:
- Creates entries in `inventory` table
- Referenced by `invoice_items`
- Stock updated automatically on invoice finalization

**API Endpoints**:
```
POST   /api/inventory                - Create product
GET    /api/inventory                - List products
GET    /api/inventory/:id            - Get details
PUT    /api/inventory/:id            - Update product
DELETE /api/inventory/:id            - Soft delete
```

**Key Fields**:
- Name, SKU (optional)
- HSN Code (for GST) - Links to government rate table
- Unit (kg, liter, piece, etc.)
- Quantity on Hand
- Purchase Price, Selling Price
- GST Rate - Auto-populated from HSN lookup, user can override
- Active status

**GST Rate Handling**:
- SmartKhata maintains HSN/GST rate lookup table
- Standard rates: 0% (essential), 5%, 12%, 18% (most), 28% (luxury/sin)
- On item creation/edit:
  - User enters HSN code
  - System auto-applies rate from master table
  - User can manually override if rate changed or unknown
- For unknown HSN: User manually enters rate (0%, 5%, 12%, 18%, or 28% only)
- Rate is stored per item for consistency

---

### Feature 4: Ledger

**What it does**: Real-time party transaction history

**Database Impact**:
- Auto-created entries in `ledger` table
- Updated on every invoice/payment
- Triggers calculate running balance

**Automatic Entries**:
- Invoice created → Credit entry (party owes money)
- Payment received → Debit entry (party paid)
- Manual adjustment → Manual entry

**API Endpoints**:
```
GET    /api/parties/:id/ledger       - Get ledger for party
GET    /api/ledger                   - Global ledger (filtered by business)
```

**Display**:
- Transaction date
- Transaction type (Invoice, Payment, Adjustment)
- Debit/Credit amount
- Running balance

---

### Feature 5: Payments

**What it does**: Record payments received or paid

**Database Impact**:
- Creates entries in `payments` table
- Triggers automatic ledger entry (debit)
- Updates party outstanding balance

**API Endpoints**:
```
POST   /api/payments                 - Record payment
GET    /api/payments                 - List payments
GET    /api/payments/:id             - Get details
PUT    /api/payments/:id             - Update payment
DELETE /api/payments/:id             - Delete payment
```

**Payment Methods Supported**:
- Cash
- Check
- Bank Transfer
- Card
- UPI
- Custom

---

## 🔗 Compliance Features

### Feature 6: E-Invoicing (GST Portal Integration)

**What it does**: Generate IRN (Invoice Reference Number) on GST portal

**Trigger**: When invoice finalized (auto or manual)

**API Calls**:
- POST to GST portal: Invoice data
- Receive: IRN, QR code, signed invoice

**Database Fields**:
- `irn` (VARCHAR 64) - Invoice reference number
- `ack_number` (VARCHAR 64) - Acknowledgement number
- `qr_code_url` (TEXT) - URL to QR image

**API Endpoints**:
```
POST   /api/invoices/:id/e-invoice   - Generate IRN
GET    /api/invoices/:id/qr-code     - Get QR image
```

**Availability**:
- DIAMOND: Limited (10/month)
- PLATINUM: 100/month
- ENTERPRISE: Unlimited

**See**: 09_INTEGRATIONS.md for GST Portal API details

---

### Feature 7: E-Way Bills

**What it does**: Generate e-way bills for goods movement

**Trigger**: Manual or auto on invoice finalization

**See**: 09_INTEGRATIONS.md for GST Portal API details

---

### Feature 8: GST Reports

**What it does**: Generate GSTR-1, GSTR-2 from invoices

**See**: Feature 17 (Reports)

---

## 💰 Accounting Features

### Feature 9: Purchase Entry

**What it does**: Record purchase invoices with three input methods

**Input Methods**:
1. Manual - Type details manually
2. OCR - Upload invoice image, auto-extract
3. AI-Assisted - Review extracted data with AI suggestions

**Database Impact**:
- Creates entry in `purchases` table
- Creates `purchase_items` entries
- Auto-creates ledger entry (debit)

**API Endpoints**:
```
POST   /api/purchases                - Create purchase
POST   /api/purchases/ocr            - Upload & extract image
GET    /api/purchases                - List purchases
GET    /api/purchases/:id            - Get details
PUT    /api/purchases/:id            - Update purchase
```

**OCR Service**: AWS Textract (See 09_INTEGRATIONS.md)

---

### Feature 10: Expenses

**What it does**: Track business expenses with categorization

**Categories**:
- Rent, Utilities, Salary, Marketing, Travel, etc.

**Database Impact**:
- Creates entries in `expenses` table
- Used in P&L reports

**API Endpoints**:
```
POST   /api/expenses                 - Create expense
GET    /api/expenses                 - List expenses
GET    /api/expenses/:id             - Get details
PUT    /api/expenses/:id             - Update expense
```

---

### Feature 11: Reconciliation

**What it does**: AI-powered automatic payment matching

**How it works**:
1. System fetches bank transactions (manual or auto upload)
2. AI compares with outstanding invoices
3. Suggests matches with confidence score
4. User confirms matches
5. Auto-creates ledger entries

**API Endpoints**:
```
POST   /api/reconciliation/match     - Get AI suggestions
POST   /api/reconciliation/confirm   - Confirm match
```

---

## 🔄 Sharing & Distribution Features

### Features 12-16: Email, WhatsApp, SMS, PDF, Print

**What it does**: Share invoices via multiple channels

**Email** (Feature 12):
- Send invoice PDF via email
- Customizable template
- Tracking (opens, downloads)

**WhatsApp** (Feature 13):
- Share invoice PDF link via WhatsApp
- Automatic message

**SMS** (Feature 14):
- Send payment reminder SMS
- Custom message templates

**PDF** (Feature 15):
- Custom PDF generation
- Branded header/footer
- Logo embedding

**Print** (Feature 16):
- Normal printer support
- Thermal printer support (58mm, 80mm)

**API Endpoints** (for all):
```
POST   /api/invoices/:id/share/email      - Send via email
POST   /api/invoices/:id/share/whatsapp   - Send via WhatsApp
POST   /api/invoices/:id/share/sms        - Send SMS reminder
GET    /api/invoices/:id/pdf              - Download PDF
```

**See**: 09_INTEGRATIONS.md for email/SMS provider details

---

## 📊 Advanced Features

### Feature 17: Reports

**Types**:
- **P&L Statement** - Profitability analysis
- **Cash Flow** - Cash position and trends
- **GST Analysis** - Tax liability breakdown
- **Party Outstanding** - Who owes how much
- **Inventory** - Stock levels and turnover

**Database**: Calculated from invoices, expenses, inventory on-demand

**API Endpoints**:
```
GET    /api/reports/profit-loss      - P&L report
GET    /api/reports/cash-flow        - Cash flow
GET    /api/reports/gst-analysis     - GST breakdown
GET    /api/reports/outstanding      - Party outstanding
GET    /api/reports/inventory        - Stock report
```

---

### Feature 18: Shared Ledger

**What it does**: Provide customer portal (read-only) to view their ledger

**Access**: Via unique login link (no password needed)

**Customer Sees**:
- Their invoices (issued, paid, overdue)
- Their payments (recorded in system)
- Outstanding balance
- Transaction history

**API Endpoints**:
```
GET    /api/shared-ledger/:token     - Get shared ledger (public)
```

---

### Features 19-20: POS, Online Store

**POS** (Feature 19):
- Quick invoice creation at counter
- Barcode scanning
- Quick pay (cash immediate)

**Online Store** (Feature 20):
- Customer can create own invoices
- Self-service ordering

**Status**: Future (V2)

---

## 🔗 Integration Features

### Features 21-24: Tally, API, Themes, Bulk Ops

**Tally Integration** (Feature 21):
- Sync data with Tally
- Bidirectional sync

**REST API** (Feature 22):
- Custom integrations
- Webhooks
- OAuth support

**Custom Themes / Branding** (Feature 23):
- Upload company logo (PNG, JPEG, WebP)
  - Stored in S3 with CDN delivery
  - Auto-used on all invoices, PDFs, emails
  - Max 2MB size, 1024x1024px recommended
- Customize colors (primary, secondary, accent)
- Custom invoice header & footer text
- Optional domain branding (ENTERPRISE only)

**Invoice Templates** (Feature 25 - NEW):
- Save reusable invoice templates
- Pre-fill: party, items, notes, payment terms
- Quick-create: 3 clicks vs manual entry
- Set as default for auto-selection
- Duplicate existing invoices as templates
- Separate templates for each party type

**Recurring Invoices** (Feature 26 - NEW):
- Auto-generate invoices on schedule (daily/weekly/monthly/quarterly/yearly)
- Set expiration date or run indefinitely
- Auto-finalize (submit to GST portal automatically)
- Auto-email (send to customer on generation)
- Pause/resume capability
- Background job runs nightly at 2 AM IST
- Perfect for subscriptions, retainers, maintenance fees

**Email Templates** (Feature 27 - NEW):
- Save custom email messages with variables
- Variables: {invoice_no}, {party_name}, {amount}, {due_date}, {payment_link}
- Categories: reminder, thank-you, urgent, invoice
- Use when sending via email sharing
- Reduce manual typing for repetitive messages
- Preview with sample data before sending

**SMS Templates** (Feature 28 - NEW):
- Save custom SMS messages (160 chars limit)
- Variables: {invoice_no}, {party_name}, {amount}, {due_date}, {payment_link}
- Categories: reminder, alert, payment, urgent
- Auto character count tracking (SMS length calculation)
- Quick-send payment reminders
- Cost optimization (compose once, send many)

**Bulk Operations** (Feature 24):
- Import invoices from CSV
- Export data to Excel
- Bulk edits
- Bulk template upload

**See**: 03_API.md for complete API endpoints, 09_INTEGRATIONS.md for GST specifications

---

## 📋 Feature Availability by Plan

| Feature | FREE | DIAMOND | PLATINUM | ENTERPRISE | CUSTOM |
|---------|------|---------|----------|-----------|--------|
| Invoicing | 15/mo | 50/mo | 200/mo | Unlimited | Custom |
| Parties | Unlimited | Unlimited | Unlimited | Unlimited | Custom |
| Inventory | 50 items | Unlimited | Unlimited | Unlimited | Custom |
| Ledger | Auto | Auto | Auto | Auto | Auto |
| Payments | Manual only | Auto | Auto | Auto | Custom |
| E-Invoicing | ❌ | 10/mo | 100/mo | Unlimited | Custom |
| E-Way Bills | ❌ | ❌ | Unlimited | Unlimited | Custom |
| GST Reports | ❌ | ❌ | Yes | Yes | Custom |
| Purchase Entry | ❌ | Manual | All methods | All | Custom |
| Expenses | ❌ | Limited | Unlimited | Unlimited | Custom |
| Reconciliation | ❌ | ❌ | Limited | Unlimited | Custom |
| Email Share | Limited | Unlimited | Unlimited | Unlimited | Custom |
| WhatsApp Share | ❌ | Limited | Unlimited | Unlimited | Custom |
| SMS Share | ❌ | 10/mo | Unlimited | Unlimited | Custom |
| PDF | Basic | Custom | Custom | Custom | Custom |
| Payment Portal | ❌ | Limited | Unlimited | Unlimited | Custom |
| Auto-Reminders | ❌ | Limited | Unlimited | Unlimited | Custom |
| Analytics | ❌ | Basic | Full | Full | Custom |
| Reports | Basic | All | All | All | Custom |

---

## 🔴 CRITICAL OWNER FEATURES (Newly Added)

**Payment Portal** (Feature 25 - CRITICAL):
- Customers pay online via unique payment link
- QR code generation (owner can upload custom QR or auto-generate)
- Accepts: UPI, Credit/Debit card, Bank transfer, PayPal
- Payment confirmation auto-sent
- Auto-marks invoice as paid
- No intermediary: Direct payment to business bank account
- Mobile-friendly payment page

**Auto-Reminders** (Feature 26 - CRITICAL):
- Scheduled reminders 2 days before due date (auto-send)
- Day-of-due reminder (auto-send)
- Overdue reminders: 1 day, 5 days, 10 days, 30 days late
- Escalating tone (friendly → urgent → legal)
- SMS + Email options
- Templates customizable per business

**Overdue Dashboard** (Feature 27 - CRITICAL):
- Real-time overdue amount (₹XXX overdue)
- Customer-wise breakdown (who owes what)
- Days late visualization
- Export list for collection agency
- Automated dunning sequence (reminder → warning → escalation)

**Analytics Dashboard** (Feature 28 - CRITICAL):
- Revenue this month vs last month (graph)
- Cash flow forecast (when money coming in)
- Top 5 customers by revenue
- Top 5 slow payers (days overdue)
- Collection rate (% of invoices paid on time)
- Average payment delay (days)
- Aging report: 0-30 / 30-60 / 60-90 / 90+ days overdue

**Payment Terms** (Feature 29):
- Define payment terms (NET 30, NET 60, etc)
- Discount if paid early: "2% off if paid in 10 days"
- Auto-calculate discount based on payment date
- Recurring invoices use default terms

**Multi-Staff Management** (Feature 30):
- Staff roles: Admin, Manager, Accountant, Viewer
- Permissions: Create, Edit, Approve, Delete, View
- Approval workflow: Create → Review → Approve → Send
- Activity log (who did what, when)
- Cannot delete (only soft delete with audit trail)

**Payment Tracking** (Feature 31):
- Track payment status per invoice
- See partial payments
- Refund capability
- Payment method recorded (UPI/Card/Bank)
- Receipt generation

**Customer Portal** (Feature 32):
- Full self-service access for customers
- View all invoices and balances
- Download invoices/receipts
- Make online payments
- Track payment history
- View aging of receivables

---

## 🚀 Implementation Phases

**Phase 1 (MVP - Week 1-4)**: Invoicing, Parties, Inventory, Ledger, Payments

**Phase 2 (Compliance - Week 5-8)**: E-Invoicing, E-Way, GST Reports

**Phase 3 (Sharing - Week 9-12)**: Email, WhatsApp, SMS, PDF, Print

---

## ⭐ Feature 33: Customer Discovery & Network Effect (CRITICAL - NEW)

**What it does**: Create a network where owners search for customers and customers auto-register when tagged, with instant notifications driving engagement

**Business Problem**:
- Owners waste time manually creating customer records
- Customers don't get visibility into their outstanding balance
- No incentive for customers to engage (transparency gap)
- System is closed (owner-only), not a network

**Solution**:
1. **Owner searches** for customer by name/phone/email → finds or creates
2. **Customer auto-registers** when invoice created (if not found)
3. **Customer gets SMS/Email notification** instantly with payment link
4. **Customer views portal** to see balance, payment history
5. **Network effect**: More customers engaged = more transparency = more repeat users

**Key Use Cases**:
- Fertilizer shop owner: Search for farmer "Ramesh" → not found → creates customer → sends GST invoice → Ramesh gets SMS → Ramesh sees his balance
- Book seller: Individual buys books → records invoice on his name → sends SMS with invoice → Individual sees his payment history
- Fuel station: Individual fills fuel → records customer → sends receipt → Individual has transparent transaction history
- Khata transparency: Every customer becomes their own ledger keeper

**Database Impact**:
- Creates entry in `customers` table (auto-generated)
- Creates notification in `customer_notifications` table
- Tracks `customer_engagement_metrics` for analytics
- Customer gets phone-verified login access

**API Endpoints**:
```
GET    /api/customers/search              - Search by name/phone/email
POST   /api/customers/auto-register       - Create if not found
POST   /api/customers/send-invoice-sms    - Notify customer
GET    /api/customer-portal/invoices      - Customer views their invoices
GET    /api/customer-portal/balance       - Customer sees outstanding
GET    /api/analytics/customer-network    - Owner sees engagement metrics
```

**Mobile UX**:
- Owner screen: "Search Customer" → Shows suggestions → "Create & Send Invoice"
- Customer screen: "SmartKhata Notification" → Click → See invoice → Pay → Get receipt

**Success Metrics**:
- 80%+ customer registration completion
- 50%+ customer portal login rate (within 30 days)
- 3x increase in payment velocity (customers who see notification pay faster)
- Network grows exponentially (customers become repeat users)

**Business Impact**:
- **Cash Flow**: Faster collection (customers who get SMS pay in 3-5 days vs 20 days)
- **Transparency**: Complete khata visibility (both sides)
- **Network Effect**: System becomes more valuable as more customers join
- **Competitive Advantage**: Khatabook-style network (beats standalone alternatives)
- **Revenue Opportunity**: Premium features for customers (invoice exports, bulk payment tracking)

---

## ⭐ Feature 34: Purchase Orders & Advanced Invoicing (NEW)

**What it does**: Create professional invoices and POs with complete customization - bank details, logo, multiple payment QR codes (UPI, Phone Pay, Bank Transfer)

**Business Use Cases**:
- Manufacturing company sends PO to supplier with bank details + UPI QR
- Retailer sends invoice to wholesaler with logo + payment QRs
- Service provider sends invoice to corporate client with bank transfer details
- NGO sends receipt with donation payment options

**Key Features**:
1. **PO Generation** - Professional purchase order template
2. **Invoice Customization** - Bank details, logo, multiple QR codes
3. **Multiple Payment Methods** - UPI QR + Phone Pay QR + Bank Transfer details
4. **Professional Templates** - Pre-built layouts with all fields
5. **Custom Branding** - Logo, colors, company letterhead
6. **Tax Configuration** - GST, HSN, item-wise tax rates
7. **Terms & Conditions** - Custom T&C on documents
8. **Payment Instructions** - Multiple payment options displayed

**Database Impact**:
- New table: `invoice_bank_details` (bank account, IFSC, UPI ID, Phone Pay ID)
- New table: `po_master` (PO tracking)
- New table: `payment_qr_codes` (multiple QR storage - UPI, Phone Pay, etc.)
- Updates to `invoices` table: Add PO reference, document type (Invoice vs PO)

**API Endpoints**:
```
POST   /api/invoices/generate-po           - Create PO
POST   /api/invoices/{id}/update-bank-details - Add bank info
POST   /api/invoices/{id}/generate-qr-codes   - Generate all payment QRs
GET    /api/invoices/{id}/as-pdf           - Download invoice/PO with all details
```

**Mobile UX**:
- New screen: "Invoice/PO Settings" (bank details, logos, payment options)
- New section: "Create PO" (same as invoice but PO template)
- Invoice preview showing: Logo + Bank details + All 3 payment QRs

**Success Metrics**:
- 100% of invoices include payment details
- Bank transfer adoption: 20%+ (B2B payments)
- Phone Pay adoption: 30%+ (modern payments)
- PO generation: 15% of B2B transactions

---

**Phase 4 (CRITICAL - Week 13-20)**: Payment Portal, Auto-Reminders, Analytics, Dashboard, Customer Discovery

**Phase 5 (Advanced - Week 21+)**: Payment Terms, Multi-Staff, Customer Portal, Reporting, PO & Advanced Invoicing

**Phase 4 (Accounting - Week 13-16)**: Purchase, Expenses, Reconciliation

**Phase 5 (Advanced - Week 17-20)**: Reports, Shared Ledger, Themes, API

---

**Related Documents**: 01_ARCHITECTURE.md, 03_API.md, 05_BUSINESS.md, 08_UX.md, 09_INTEGRATIONS.md
