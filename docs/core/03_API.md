# 3. SmartKhata - REST API Specification

**Purpose**: Complete REST API documentation  
**Audience**: Backend developers, mobile developers, integration partners  
**Consolidates**: API_SPECIFICATION.md

---

## 🔐 Authentication

### JWT-Based Authentication

**Login Endpoint**:
```
POST /api/auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response 200:
{
  "access_token": "eyJhbGc...",
  "refresh_token": "eyJhbGc...",
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "business_id": "uuid",
    "role": "owner"
  }
}
```

**Headers for All Requests**:
```
Authorization: Bearer {access_token}
X-Business-ID: {business_id}
```

---

## 📋 Core Endpoints

### 1. Parties API

```
GET    /api/parties                      - List all parties
POST   /api/parties                      - Create party
GET    /api/parties/:id                  - Get party details
PUT    /api/parties/:id                  - Update party
DELETE /api/parties/:id                  - Soft delete
GET    /api/parties/:id/ledger           - Get party ledger
GET    /api/parties/:id/outstanding      - Get outstanding amount
```

**Example: Create Party**:
```json
POST /api/parties
{
  "name": "ABC Retail Store",
  "type": "customer",
  "email": "abc@retail.com",
  "phone": "9876543210",
  "gst_number": "27AABCT1234A1Z5",
  "address": "123 Main St, Mumbai"
}
```

---

### 2. Invoices API

```
POST   /api/invoices                     - Create invoice (draft)
GET    /api/invoices                     - List invoices
GET    /api/invoices/:id                 - Get invoice details
PUT    /api/invoices/:id                 - Update invoice (draft only)
DELETE /api/invoices/:id                 - Delete draft invoice
POST   /api/invoices/:id/finalize        - Finalize invoice
POST   /api/invoices/:id/duplicate       - Duplicate invoice
GET    /api/invoices/:id/pdf             - Download PDF
POST   /api/invoices/:id/share/email     - Send via email
POST   /api/invoices/:id/share/whatsapp  - Send via WhatsApp
GET    /api/invoices/number/:number      - Get by invoice number
```

**Example: Create Invoice**:
```json
POST /api/invoices
{
  "party_id": "uuid",
  "invoice_date": "2026-01-19",
  "due_date": "2026-02-19",
  "items": [
    {
      "inventory_id": "uuid",
      "quantity": 10,
      "unit_price": 100,
      "gst_rate": 18
    }
  ],
}
```

---

### 2B. Purchase Orders & Advanced Invoicing API

```
POST   /api/po                           - Create purchase order
GET    /api/po                           - List purchase orders
GET    /api/po/:id                       - Get PO details
PUT    /api/po/:id                       - Update PO (draft only)
POST   /api/po/:id/send                  - Send PO to supplier
POST   /api/po/:id/accept                - Supplier accepts PO
POST   /api/po/:id/reject                - Supplier rejects PO
POST   /api/po/:id/convert-to-invoice    - Convert PO to invoice
GET    /api/po/:id/pdf                   - Download PO as PDF
POST   /api/po/:id/share/email           - Send PO via email
```

**Bank Details Management**:
```
POST   /api/business/bank-details        - Add bank account
GET    /api/business/bank-details        - List all bank accounts
PUT    /api/business/bank-details/:id    - Update bank account
DELETE /api/business/bank-details/:id    - Delete bank account
POST   /api/business/bank-details/:id/set-default - Set as default
POST   /api/business/bank-details/:id/generate-qr - Generate QR codes
```

**Invoice Customization**:
```
POST   /api/invoices/:id/customize       - Customize invoice (add bank, logo, terms)
POST   /api/invoices/:id/generate-qr    - Generate payment QR codes
GET    /api/invoices/:id/payment-qr     - Get all QR codes for invoice
```

**Example: Create Purchase Order**:
```json
POST /api/po
{
  "po_number": "PO-00123",
  "po_date": "2026-01-19",
  "supplier_id": "uuid",
  "delivery_date": "2026-02-19",
  "items": [
    {
      "description": "Raw Material A",
      "quantity": 100,
      "unit": "kg",
      "unit_price": 500,
      "tax_rate": 5
    }
  ],
  "payment_terms": "30 days",
  "notes": "Urgent delivery required"
}
```

**Example: Add Bank Details**:
```json
POST /api/business/bank-details
{
  "account_holder_name": "ABC Enterprises",
  "account_number": "123456789012",
  "ifsc_code": "HDFC0001234",
  "bank_name": "HDFC Bank",
  "upi_id": "abcshop@upi",
  "phone_pay_id": "9876543210@phonepe",
  "is_default": true
}
```

**Example: Generate Invoice with QR Codes**:
```json
POST /api/invoices/:id/generate-qr
{
  "qr_types": ["upi", "phone_pay", "bank_transfer"],
  "amount": 10000
}

Response:
{
  "qr_codes": [
    {
      "qr_type": "upi",
      "qr_code_url": "https://s3.../upi-qr.png",
      "amount": 10000
    },
    {
      "qr_type": "phone_pay",
      "qr_code_url": "https://s3.../phonepe-qr.png"
    },
    {
      "qr_type": "bank_transfer",
      "qr_code_url": "https://s3.../bank-qr.png",
      "account_number": "123456789012",
      "ifsc": "HDFC0001234"
    }
  ]
}
```

---

### 3. Inventory API

```
POST   /api/inventory                    - Create product
GET    /api/inventory                    - List products
GET    /api/inventory/:id                - Get product details
PUT    /api/inventory/:id                - Update product
DELETE /api/inventory/:id                - Soft delete
```

---

### 4. Payments API

```
POST   /api/payments                     - Record payment
GET    /api/payments                     - List payments
GET    /api/payments/:id                 - Get payment details
PUT    /api/payments/:id                 - Update payment
DELETE /api/payments/:id                 - Delete payment
GET    /api/parties/:id/payments         - Get payments for party
```

---

### 5. Ledger API

```
GET    /api/ledger                       - Global ledger (paginated)
GET    /api/parties/:id/ledger           - Party ledger
GET    /api/ledger/:id                   - Get ledger entry details
```

---

### 6. Reports API

```
GET    /api/reports/profit-loss          - P&L report
GET    /api/reports/cash-flow            - Cash flow report
GET    /api/reports/gst-analysis         - GST breakdown
GET    /api/reports/outstanding          - Outstanding report
GET    /api/reports/inventory            - Inventory report
```

**Query Parameters**:
```
?start_date=2026-01-01&end_date=2026-01-31
?group_by=party|category|month
?format=json|csv|pdf
```

---

### 7. Purchase API

```
POST   /api/purchases                    - Create purchase
GET    /api/purchases                    - List purchases
POST   /api/purchases/ocr                - Upload & extract via OCR
GET    /api/purchases/:id                - Get purchase details
PUT    /api/purchases/:id                - Update purchase
```

---

### 8. Expenses API

```
POST   /api/expenses                     - Create expense
GET    /api/expenses                     - List expenses
GET    /api/expenses/:id                 - Get expense details
PUT    /api/expenses/:id                 - Update expense
DELETE /api/expenses/:id                 - Delete expense
```

---

### 9. E-Invoicing API

```
POST   /api/invoices/:id/e-invoice       - Generate IRN
GET    /api/invoices/:id/qr-code         - Get QR image
GET    /api/invoices/:id/e-invoice/status - Check IRN status
```

---

### 10. Reconciliation API

```
POST   /api/reconciliation/match         - Get AI match suggestions
POST   /api/reconciliation/confirm       - Confirm payment match
GET    /api/reconciliation/status        - Get reconciliation status
```

### 11. Expense Splitting API (FREE)

```
POST   /api/expense-groups                 - Create expense group
GET    /api/expense-groups                 - List user's groups
GET    /api/expense-groups/:id             - Get group details
PUT    /api/expense-groups/:id             - Update group
DELETE /api/expense-groups/:id             - Delete group

POST   /api/expense-groups/:id/members     - Add members to group
DELETE /api/expense-groups/:id/members/:user_id - Remove member

POST   /api/expense-groups/:id/expenses    - Add expense to group
GET    /api/expense-groups/:id/expenses    - List group expenses
PUT    /api/expense-groups/:id/expenses/:expense_id - Update expense
DELETE /api/expense-groups/:id/expenses/:expense_id - Delete expense

POST   /api/expense-groups/:id/splits      - Configure expense splits
GET    /api/expense-groups/:id/splits      - Get split configurations

GET    /api/expense-groups/:id/balances    - Get group balances
POST   /api/expense-groups/:id/settle      - Record settlement payment
GET    /api/expense-groups/:id/activity    - Get group activity feed

POST   /api/personal-ledger                - Add personal ledger entry
GET    /api/personal-ledger                - Get personal balances
GET    /api/personal-ledger/:contact_id    - Get balance with specific contact
POST   /api/personal-ledger/:contact_id/settle - Record settlement
DELETE /api/personal-ledger/:contact_id    - Remove contact from ledger

POST   /api/invite                         - Invite friend to platform
GET    /api/invite                         - List sent invitations
POST   /api/invite/:id/resend              - Resend invitation
```

**Split Types**:
- `equal`: Divide equally among members
- `percentage`: Custom percentages per member
- `exact`: Exact amounts per member
- `shares`: Split based on number of shares

**Personal Ledger Types**:
- `i_owe`: I owe this person money
- `owes_me`: This person owes me money

**Example: Create Group Expense**:
```
POST /api/expense-groups/123/expenses
{
  "description": "Dinner at Restaurant",
  "amount": 1200.00,
  "currency": "INR",
  "category": "Food",
  "split_type": "equal",
  "paid_by": "user-456",
  "split_among": ["user-456", "user-789", "user-101"]
}
```

**Example: Add Personal Ledger Entry**:
```
POST /api/personal-ledger
{
  "contact_name": "John Doe",
  "contact_phone": "+91-9876543210",
  "amount": 500.00,
  "type": "owes_me",
  "description": "Movie tickets",
  "due_date": "2026-02-01"
}
```

### 12. Digital Bahi Khata API (FREE)

```
POST   /api/digital-khata                 - Create new khata book
GET    /api/digital-khata                 - List user's khata books
GET    /api/digital-khata/:id             - Get khata book details
PUT    /api/digital-khata/:id             - Update khata book
DELETE /api/digital-khata/:id             - Delete khata book

POST   /api/digital-khata/:id/transactions - Add transaction (credit/debit)
GET    /api/digital-khata/:id/transactions - List transactions
GET    /api/digital-khata/:id/transactions/:tid - Get transaction details
PUT    /api/digital-khata/:id/transactions/:tid - Update transaction
DELETE /api/digital-khata/:id/transactions/:tid - Delete transaction

POST   /api/digital-khata/:id/parties      - Add party (customer/supplier)
GET    /api/digital-khata/:id/parties      - List parties
GET    /api/digital-khata/:id/parties/:pid - Get party details
PUT    /api/digital-khata/:id/parties/:pid - Update party

GET    /api/digital-khata/:id/balance      - Get outstanding balance
POST   /api/digital-khata/:id/reminders    - Send payment reminder
POST   /api/digital-khata/:id/voice-entry  - Voice-to-text transaction
GET    /api/digital-khata/:id/reports      - Get basic reports
```

**Transaction Types**:
- `credit`: Money given (diye) - increases outstanding
- `debit`: Money received (liye) - decreases outstanding

**Voice Input Languages**:
- `hi`: Hindi
- `en`: English
- `hi-en`: Mixed Hindi-English

**Example: Add Khata Transaction**:
```
POST /api/digital-khata/123/transactions
{
  "party_id": "party-456",
  "type": "credit",
  "amount": 2500.00,
  "description": "Rice and dal delivery",
  "category": "goods",
  "transaction_date": "2026-01-20"
}
```

**Example: Voice Entry**:
```
POST /api/digital-khata/123/voice-entry
{
  "audio_data": "base64-encoded-audio",
  "language": "hi",
  "party_context": "Ramu Kirana"
}
```

---

## 🔄 Pagination & Filtering

**Pagination**:
```
GET /api/invoices?page=1&limit=20&sort=-created_at

Response:
{
  "data": [...],
  "total": 100,
  "page": 1,
  "limit": 20,
  "pages": 5
}
```

**Filtering**:
```
GET /api/invoices?status=paid&party_id=uuid&date_from=2026-01-01
GET /api/parties?type=customer&search=abc
```

---

## ✅ Response Format

**Success (200)**:
```json
{
  "success": true,
  "data": {...},
  "message": "Operation successful"
}
```

**List Success (200)**:
```json
{
  "success": true,
  "data": [...],
  "total": 100,
  "page": 1
}
```

**Error (400/401/403/404/500)**:
```json
{
  "success": false,
  "error": "error_code",
  "message": "Human-readable error message",
  "details": {...}
}
```

---

## 🔐 Permission Matrix

**Permission Checks** (applied by middleware):

| Endpoint | Owner | Manager | Staff |
|----------|-------|---------|-------|
| POST /invoices | ✅ | ✅ | ✅ |
| PUT /invoices | ✅ | ✅ | Own only |
| DELETE /invoices | ✅ | ✅ | Own only |
| GET /reports | ✅ | ✅ | ❌ |
| POST /parties | ✅ | ✅ | ✅ |
| GET /ledger | ✅ | ✅ | Own party |
| POST /payments | ✅ | ✅ | ✅ |

---

## ⚙️ Business Settings API

### Branding & Customization
```
GET    /api/settings/branding           - Get logo, colors, branding
PUT    /api/settings/branding           - Update branding (colors, footer, header)
POST   /api/settings/branding/logo      - Upload logo to S3
DELETE /api/settings/branding/logo      - Delete logo

GET    /api/settings/defaults           - Get invoice defaults
PUT    /api/settings/defaults           - Update prefix, number, payment terms
PUT    /api/settings/invoice-footer     - Update custom footer
PUT    /api/settings/invoice-header     - Update custom header
```

**Example: Update Branding**:
```json
PUT /api/settings/branding
{
  "primary_color": "#1976D2",
  "secondary_color": "#DC004E",
  "accent_color": "#4CAF50",
  "invoice_footer": "Thank you for your business!",
  "default_payment_terms": "NET 30"
}
```

**Example: Upload Logo**:
```
POST /api/settings/branding/logo
Content-Type: multipart/form-data

file: [image.png]

Response 200:
{
  "logo_url": "https://cdn.smartkhata.com/logos/business_123/logo.png",
  "uploaded_at": "2026-01-19T10:30:00Z"
}
```

---

## 📝 Invoice Templates API

### Save & Reuse Templates
```
GET    /api/templates/invoices          - List all invoice templates
POST   /api/templates/invoices          - Create new template
GET    /api/templates/invoices/:id      - Get template details
PUT    /api/templates/invoices/:id      - Update template
DELETE /api/templates/invoices/:id      - Delete template
POST   /api/templates/invoices/:id/use  - Use template (pre-fill invoice)
POST   /api/templates/invoices/:id/set-default - Mark as default
```

**Example: Create Template**:
```json
POST /api/templates/invoices
{
  "name": "Standard Invoice",
  "description": "Default template for all customers",
  "default_party_id": "uuid",
  "default_notes": "Payment by bank transfer only",
  "template_items": [
    {
      "inventory_id": "uuid",
      "quantity": 1,
      "rate": 100,
      "gst_rate": 18
    }
  ],
  "is_default": true
}
```

---

## 📧 Email Templates API

### Reusable Email Messages
```
GET    /api/templates/emails            - List all email templates
POST   /api/templates/emails            - Create new email template
PUT    /api/templates/emails/:id        - Update email template
DELETE /api/templates/emails/:id        - Delete email template
POST   /api/templates/emails/:id/preview - Preview with sample data
```

**Example: Create Email Template**:
```json
POST /api/templates/emails
{
  "name": "Invoice Reminder",
  "subject": "Payment Reminder - Invoice {invoice_no}",
  "body": "Hi {party_name},<br>Please pay <b>{amount}</b> by {due_date}",
  "category": "reminder",
  "is_default": false
}
```

**Supported Variables**: `{invoice_no}`, `{party_name}`, `{amount}`, `{due_date}`, `{payment_link}`

---

## 💬 SMS Templates API

### Reusable SMS Messages
```
GET    /api/templates/sms               - List all SMS templates
POST   /api/templates/sms               - Create new SMS template
PUT    /api/templates/sms/:id           - Update SMS template
DELETE /api/templates/sms/:id           - Delete SMS template
GET    /api/templates/sms/:id/character-count - Get SMS length
```

**Example: Create SMS Template**:
```json
POST /api/templates/sms
{
  "name": "Payment Reminder",
  "message": "Hi {party_name}, Please pay {amount} by {due_date}. Link: {payment_link}",
  "category": "reminder",
  "is_default": false
}
```

**Note**: SMS has 160-character limit per message. Variables auto-calculated in length.

---

## 🔄 Recurring Invoices API

### Auto-Billing Setup
```
GET    /api/recurring-invoices          - List all recurring invoices
POST   /api/recurring-invoices          - Create new recurring invoice
GET    /api/recurring-invoices/:id      - Get details
PUT    /api/recurring-invoices/:id      - Update recurring invoice
DELETE /api/recurring-invoices/:id      - Delete recurring invoice
POST   /api/recurring-invoices/:id/pause - Pause (don't generate next)
POST   /api/recurring-invoices/:id/resume - Resume paused recurring
POST   /api/recurring-invoices/generate - Cron: Generate due invoices
```

**Example: Create Recurring Invoice**:
```json
POST /api/recurring-invoices
{
  "template_id": "uuid",
  "party_id": "uuid",
  "frequency": "monthly",
  "start_date": "2026-02-01",
  "end_date": "2027-02-01",
  "auto_finalize": true,
  "auto_email": true,
  "is_active": true
}
```

**Frequency Options**: `daily`, `weekly`, `monthly`, `quarterly`, `yearly`

**Auto-Generation Logic**:
- Runs nightly at 2 AM IST
- Generates invoice if next_invoice_date <= TODAY
- Auto-finalizes if flag enabled
- Sends email to party if auto_email enabled
- Updates next_invoice_date based on frequency

---

## � PAYMENT PORTAL API (CRITICAL - Customer Payments)

### Payment Link Generation
```
GET    /api/payments/links/:invoice_id        - Get payment link for invoice
POST   /api/payments/links/generate            - Generate new payment link
POST   /api/payments/links/:id/qrcode          - Generate QR code
POST   /api/payments/links/:id/upload-qrcode   - Upload custom QR code
DELETE /api/payments/links/:id                 - Deactivate link
```

**Example: Generate Payment Link**:
```json
POST /api/payments/links/generate
{
  "invoice_id": "uuid",
  "accepted_methods": ["upi", "card", "netbanking"],
  "expires_in_days": 30
}

Response 200:
{
  "payment_link": "https://smartkhata.com/pay/xyz123abc",
  "qr_code_url": "https://cdn.smartkhata.com/qr/xyz123abc.png",
  "expires_at": "2026-02-19T00:00:00Z"
}
```

**Example: Upload Custom QR Code**:
```
POST /api/payments/links/xyz123abc/upload-qrcode
Content-Type: multipart/form-data

file: [qrcode.png]

Response 200:
{
  "qr_code_url": "https://cdn.smartkhata.com/qr/xyz123abc.png",
  "uploaded_at": "2026-01-19T10:30:00Z"
}
```

---

## 📱 PAYMENT PORTAL (Customer Facing)

### Payment Page (Public, No Auth Needed)
```
GET    /pay/:payment_link_token              - Public payment page (customer views)
POST   /pay/:payment_link_token/process      - Process payment (Razorpay webhook)
```

**Payment Page Flow**:
1. Customer receives email with link: `smartkhata.com/pay/xyz123abc`
2. Opens link → Sees invoice amount + QR code
3. Can pay via:
   - UPI (scan QR or tap button)
   - Card/Debit
   - Net Banking
   - Wallet
4. Success → Confirmation email sent
5. Invoice auto-marked as paid

---

## 🔔 AUTO-REMINDERS API

### Reminder Setup & Tracking
```
GET    /api/reminders/schedule               - Get reminder schedule for invoice
POST   /api/reminders/schedule               - Create reminder schedule
GET    /api/reminders/history/:invoice_id    - View sent reminders
POST   /api/reminders/send-now/:invoice_id   - Manually send reminder now
```

**Example: Setup Auto-Reminders**:
```json
POST /api/reminders/schedule
{
  "invoice_id": "uuid",
  "reminders": [
    {
      "type": "pre_due",
      "days_before_due": 2,
      "channel": "email",
      "template_id": "uuid"
    },
    {
      "type": "due",
      "days_before_due": 0,
      "channel": "sms",
      "template_id": "uuid"
    },
    {
      "type": "overdue_1day",
      "days_after_due": 1,
      "channel": "sms",
      "template_id": "uuid"
    }
  ]
}
```

**Reminder Types**: `pre_due`, `due`, `overdue_1day`, `overdue_5day`, `overdue_10day`, `overdue_30day`

---

## 📊 DASHBOARD & ANALYTICS API

### Overdue Dashboard
```
GET    /api/dashboard/overdue                - Get overdue invoices summary
GET    /api/dashboard/overdue/breakdown      - Breakdown by customer
GET    /api/dashboard/aging-report           - 30/60/90+ days aging
```

**Example Response**:
```json
GET /api/dashboard/overdue
{
  "total_overdue_amount": 125000,
  "overdue_invoice_count": 12,
  "top_slow_payers": [
    {
      "customer_name": "ABC Store",
      "amount_due": 50000,
      "days_overdue": 45,
      "payment_link": "smartkhata.com/pay/xyz"
    }
  ],
  "aging": {
    "0_30_days": 30000,
    "30_60_days": 45000,
    "60_90_days": 35000,
    "90_plus_days": 15000
  }
}
```

### Analytics Dashboard
```
GET    /api/dashboard/analytics              - Get full analytics dashboard
GET    /api/dashboard/revenue-trend          - Revenue this vs last month
GET    /api/dashboard/collection-rate        - % of invoices paid on time
GET    /api/dashboard/top-customers          - Top 5 customers by revenue
GET    /api/dashboard/slow-payers            - Top 5 overdue customers
```

**Example Response**:
```json
GET /api/dashboard/analytics
{
  "period": "this_month",
  "total_invoiced": 500000,
  "total_received": 350000,
  "total_overdue": 150000,
  "collection_rate": 78,
  "avg_payment_delay": 5,
  "cash_flow_forecast": {
    "tomorrow": 25000,
    "this_week": 75000,
    "next_week": 100000
  }
}
```

---

## 💰 PAYMENT TERMS & DISCOUNTS API

### Payment Terms Management
```
GET    /api/payment-terms                    - List all payment terms
POST   /api/payment-terms                    - Create new payment term
PUT    /api/payment-terms/:id                - Update term
DELETE /api/payment-terms/:id                - Delete term
POST   /api/payment-terms/:id/set-default    - Set as default
```

**Example: Create Payment Term**:
```json
POST /api/payment-terms
{
  "name": "2% Discount in 10 Days",
  "days_until_due": 30,
  "early_payment_discount": 2,
  "early_payment_days": 10,
  "is_default": true
}
```

**Auto-Discount Logic**: If paid within 10 days → 2% discount applied automatically

---

## 👥 STAFF & APPROVAL WORKFLOW API

### Role-Based Access
```
GET    /api/staff/roles                      - List staff roles
PUT    /api/staff/:id/role                   - Assign role to staff
GET    /api/staff/:id/permissions            - Get staff permissions
```

### Approval Workflow
```
GET    /api/approvals/pending                - Get invoices pending approval
POST   /api/approvals/:invoice_id/approve    - Approve invoice
POST   /api/approvals/:invoice_id/reject     - Reject invoice
GET    /api/approvals/history/:invoice_id    - View approval trail
```

**Roles & Permissions**:
- **Admin**: Create, Edit, Approve, Delete, View, Reports
- **Manager**: Create, Edit, Approve, View, Reports
- **Accountant**: Create, Edit, View, Reports (no approval/delete)
- **Viewer**: View only (read-only access)

---

## �🔄 Rate Limiting

**Tier 1 (FREE)**:
- 100 requests/hour
- 1000 requests/day

**Tier 2 (DIAMOND+)**:
- 1000 requests/hour
- 10000 requests/day

**Headers**:
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 997
X-RateLimit-Reset: 1642624800
```

---

## 📱 Mobile App API Client

**Location**: `MOBILE/src/api/client.ts`

```typescript
const apiClient = {
  // Auth
  login: (email, password) => POST('/auth/login', {email, password}),
---

## 🌐 Customer Discovery & Network API (CRITICAL - NEW)

### Owner-Side: Customer Search & Auto-Register

```
GET /api/customers/search
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

Query Params:
  - q: "Ramesh" OR "+919876543210" OR "ramesh@email.com"
  - limit: 10

Response 200:
{
  "results": [
    {
      "customer_id": "uuid-123",
      "name": "Ramesh Kumar",
      "phone": "+919876543210",
      "type": "individual",
      "status": "registered",
      "portal_verified": true,
      "last_invoice": "2025-01-15",
      "total_balance": 5000
    },
    {
      "customer_id": "NEW",
      "name": "Ramesh (Create New)",
      "phone": "+919876543210",
      "type": "individual",
      "status": "not_found",
      "action": "create_and_send"
    }
  ]
}
```

```
POST /api/customers/auto-register
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

{
  "name": "Ramesh Kumar",
  "phone": "+919876543210",
  "email": "ramesh@email.com",
  "profile_type": "individual",  -- or "farmer", "retailer", "business"
  "auto_send_notification": true
}

Response 201:
{
  "customer_id": "uuid-456",
  "name": "Ramesh Kumar",
  "phone_verified": false,
  "email_verified": false,
  "otp_sent": true,
  "message": "Customer created. OTP sent to +919876543210"
}
```

### Send Invoice to Customer (Triggers Notification)

```
POST /api/invoices/{invoice_id}/send-to-customer
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

{
  "customer_id": "uuid-456",
  "channels": ["sms", "email"],  -- Which to send
  "message": "Your fertilizer invoice is ready. Pay now: {payment_link}",
  "auto_register": true  -- If customer not in system yet
}

Response 200:
{
  "notification_id": "uuid-789",
  "invoice_id": "uuid-456",
  "channels_sent": ["sms", "email"],
  "sms_status": "sent",
  "email_status": "sent",
  "message": "Invoice sent to Ramesh Kumar"
}
```

### Owner Dashboard: Network Engagement

```
GET /api/analytics/customer-network
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

Response 200:
{
  "total_customers": 1250,
  "verified_customers": 980,          -- Phone/Email verified
  "active_customers_7d": 450,         -- Login in last 7 days
  "notification_open_rate": 68,       -- % of emails opened
  "avg_payment_days": 5,              -- Average payment timeline
  "repeat_customer_rate": 72,         -- % with 2+ invoices
  "payment_velocity_impact": 35,      -- % faster payment vs non-notified
  "monthly_trend": {
    "new_customers": 120,
    "verified": 95,
    "portal_logins": 340,
    "invoices_sent": 850,
    "notifications_sent": 890
  }
}
```

---

### Customer-Side: Registration & Portal

```
POST /api/customer-portal/register
(No Auth - Public endpoint)

{
  "phone": "+919876543210",
  "otp": "123456"
}

Response 200:
{
  "customer_id": "uuid-456",
  "customer_name": "Ramesh Kumar",
  "token": "eyJhbGc...",
  "businesses": [
    {
      "business_id": "uuid-111",
      "business_name": "Fertilizer Shop",
      "total_balance": 5000,
      "last_invoice": "2025-01-15"
    }
  ]
}
```

```
GET /api/customer-portal/invoices
Authorization: Bearer {customer_token}

Query Params:
  - business_id: "uuid-111" (if customer works with multiple businesses)
  - status: "paid|unpaid|all"

Response 200:
{
  "invoices": [
    {
      "invoice_id": "uuid-555",
      "business_name": "Fertilizer Shop",
      "date": "2025-01-15",
      "amount": 2500,
      "status": "paid",
      "payment_date": "2025-01-16",
      "pdf_url": "https://s3.../invoice.pdf"
    },
    {
      "invoice_id": "uuid-556",
      "business_name": "Fertilizer Shop",
      "date": "2025-01-20",
      "amount": 2500,
      "status": "unpaid",
      "due_date": "2025-02-05",
      "payment_link": "https://smartkhata.com/pay/uuid-556",
      "pdf_url": "https://s3.../invoice.pdf"
    }
  ],
  "total_outstanding": 2500,
  "total_paid": 5000
}
```

```
GET /api/customer-portal/balance
Authorization: Bearer {customer_token}
X-Business-ID: {business_id}

Response 200:
{
  "business_name": "Fertilizer Shop",
  "total_balance_outstanding": 2500,
  "total_paid_to_date": 5000,
  "transaction_count": 4,
  "last_transaction": "2025-01-20",
  "payment_link": "https://smartkhata.com/pay/business-uuid-111",
  "payment_methods": ["upi", "card", "netbanking"]
}
```

---

### Notification System (Background Jobs)

```
POST /api/notifications/send-batch
(Internal - Triggered by scheduler)

{
  "type": "invoice_created",
  "business_id": "uuid-111",
  "filters": {
    "created_after": "2025-01-20T10:00:00Z"
  }
}

Response 200:
{
  "notifications_queued": 47,
  "channels": {
    "sms": 45,
    "email": 47
  },
  "estimated_delivery": "5-30 seconds"
}
```

```
GET /api/notifications/delivery-report
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

Query Params:
  - date_from: "2025-01-20"
  - date_to: "2025-01-21"

Response 200:
{
  "total_sent": 150,
  "successful": 148,
  "failed": 2,
  "open_rate": 68,
  "channels": {
    "sms": {
      "sent": 90,
      "delivered": 90,
      "failed": 0
    },
    "email": {
      "sent": 60,
      "delivered": 58,
      "failed": 2,
      "open_rate": 68
    }
  }
}
```

---

  
  // Invoices
  createInvoice: (data) => POST('/invoices', data),
  getInvoices: (filters) => GET('/invoices', filters),
  shareInvoice: (id, channel) => POST(`/invoices/${id}/share/${channel}`),
  
  // Parties
  getParties: (filters) => GET('/parties', filters),
  getPartyLedger: (id) => GET(`/parties/${id}/ledger`),
  
  // Customer Discovery & Network
  searchCustomer: (q) => GET('/customers/search', {q}),
  autoRegisterCustomer: (data) => POST('/customers/auto-register', data),
  sendInvoiceToCustomer: (invoiceId, data) => POST(`/invoices/${invoiceId}/send-to-customer`, data),
  getCustomerNetworkAnalytics: () => GET('/analytics/customer-network'),
  
  // Offline Sync
  syncData: () => { /* batch sync */ }
}
```

---

**Related Documents**: 01_ARCHITECTURE.md, 02_FEATURES.md, 04_SECURITY.md, 07_DEVELOPMENT.md, 09_INTEGRATIONS.md
