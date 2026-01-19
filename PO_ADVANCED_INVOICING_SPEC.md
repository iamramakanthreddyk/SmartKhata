# Feature #34: Purchase Orders & Advanced Invoicing

**Status**: Specification Complete - Ready for Implementation  
**Priority**: HIGH (Enables professional B2B operations)  
**Timeline**: Phase 5 (Advanced Features - Week 21+)  
**Owner**: Business & Finance Operations

---

## 📌 Executive Summary

**Problem**: Businesses need professional invoices and purchase orders with multiple payment options, bank details, logos, and sophisticated document generation.

**Solution**: Enable businesses to:
1. Generate professional purchase orders with supplier tracking
2. Create customized invoices with branding, bank details, and payment QR codes
3. Accept multiple payment methods (UPI, Phone Pay, Bank Transfer)
4. Track PO lifecycle from creation to invoice conversion

**Impact**: 
- Enables B2B operations (manufacturer ↔ supplier)
- Supports GST compliance with complete documentation
- Improves payment collection with multiple QR codes
- Increases professional credibility with branded documents

---

## 🎯 Feature Details

### What It Does

**Purchase Order Management**:
- Create POs with items, quantities, pricing
- Send to suppliers for acknowledgment
- Track acceptance/rejection status
- Convert accepted POs to invoices
- Store PO history for audit trail

**Advanced Invoice Customization**:
- Add bank account details
- Upload company logo/branding
- Configure payment methods (UPI, Phone Pay, Bank Transfer)
- Generate QR codes for each payment method
- Embed bank details in PDF

**Multiple Payment QR Codes**:
- UPI QR (via Razorpay API)
- Phone Pay QR (via Phone Pay API)
- Bank Transfer QR (static bank details)
- All 3 displayed on single invoice PDF

### Key Use Cases

1. **Manufacturing → Supplier**
   - Create PO with raw materials
   - Supplier receives via email + SMS notification
   - Accept/reject PO
   - Convert to invoice after delivery
   - Payment via bank transfer QR on invoice

2. **Retailer → Wholesaler**
   - Create PO with stock items
   - Track delivery dates
   - Convert to invoice upon receipt
   - Pay with UPI or Phone Pay QR

3. **Service Provider → Corporate Client**
   - Create invoice with service details + bank info
   - Client receives with multiple payment options
   - Quick payment via UPI/Phone Pay

4. **NGO → Donors**
   - Create receipt/invoice for donation
   - Multiple payment methods for donors
   - Bank transfer with custom note

---

## 🗄️ Database Schema

### Table 25: invoice_bank_details

**Purpose**: Store bank account information for invoice payments

```sql
CREATE TABLE invoice_bank_details (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  business_id UUID NOT NULL REFERENCES businesses(id) ON DELETE CASCADE,
  
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
  
  -- QR Code URLs (stored after generation)
  upi_qr_url TEXT,
  phone_pay_qr_url TEXT,
  google_pay_qr_url TEXT,
  
  -- Metadata
  is_default BOOLEAN DEFAULT true,
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_account UNIQUE(business_id, account_number),
  INDEX idx_business (business_id),
  INDEX idx_default (business_id, is_default)
);
```

**Sample Data**:
```json
{
  "id": "uuid",
  "business_id": "uuid",
  "account_holder_name": "ABC Enterprises Pvt Ltd",
  "account_number": "123456789012",
  "ifsc_code": "HDFC0001234",
  "bank_name": "HDFC Bank",
  "branch_name": "Pune",
  "upi_id": "abcshop@upi",
  "phone_pay_id": "9876543210@phonepe",
  "razorpay_account_id": "acc_abc123",
  "upi_qr_url": "https://s3.amazonaws.com/smartkhata/qr/upi/abc123.png",
  "phone_pay_qr_url": "https://s3.amazonaws.com/smartkhata/qr/phonepe/abc123.png",
  "is_default": true,
  "is_active": true
}
```

---

### Table 26: po_master

**Purpose**: Track purchase orders with supplier and status

```sql
CREATE TABLE po_master (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  business_id UUID NOT NULL REFERENCES businesses(id) ON DELETE CASCADE,
  
  -- PO Identification
  po_number VARCHAR(50) NOT NULL,        -- "PO-2026-00123"
  po_date DATE NOT NULL,
  
  -- Supplier
  supplier_id UUID NOT NULL REFERENCES parties(id) ON DELETE RESTRICT,
  
  -- Status
  po_status VARCHAR(50) DEFAULT 'draft', -- draft|issued|accepted|rejected|closed|converted
  
  -- Amounts
  total_amount DECIMAL(12,2),
  tax_amount DECIMAL(12,2),
  grand_total DECIMAL(12,2),
  
  -- Delivery & Terms
  delivery_date DATE,
  payment_terms VARCHAR(100),            -- "30 days", "COD", "Advance 50%"
  notes TEXT,                            -- Special instructions
  
  -- Conversion
  invoice_id UUID REFERENCES invoices(id),  -- When PO becomes invoice
  
  -- Audit Trail
  created_by_staff_id UUID NOT NULL REFERENCES staff(id),
  accepted_by_supplier_id UUID REFERENCES staff(id),
  accepted_at TIMESTAMP,
  rejected_at TIMESTAMP,
  rejection_reason TEXT,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  CONSTRAINT unique_po_number UNIQUE(business_id, po_number),
  INDEX idx_business (business_id),
  INDEX idx_supplier (supplier_id),
  INDEX idx_status (business_id, po_status),
  INDEX idx_created (created_at)
);
```

**Sample Data**:
```json
{
  "id": "uuid",
  "business_id": "uuid",
  "po_number": "PO-2026-00123",
  "po_date": "2026-01-20",
  "supplier_id": "uuid-of-supplier",
  "po_status": "accepted",
  "total_amount": 50000,
  "tax_amount": 2500,
  "grand_total": 52500,
  "delivery_date": "2026-02-20",
  "payment_terms": "30 days",
  "notes": "Urgent delivery required. Call if issues.",
  "accepted_at": "2026-01-22T10:30:00Z",
  "created_by_staff_id": "uuid-of-owner"
}
```

---

### Table 27: payment_qr_codes

**Purpose**: Store multiple QR codes for single invoice

```sql
CREATE TABLE payment_qr_codes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  business_id UUID NOT NULL REFERENCES businesses(id) ON DELETE CASCADE,
  invoice_id UUID NOT NULL REFERENCES invoices(id) ON DELETE CASCADE,
  
  -- QR Type
  qr_type VARCHAR(50) NOT NULL,          -- upi|phone_pay|google_pay|bank_transfer
  
  -- QR Data
  qr_code_url TEXT NOT NULL,             -- S3 URL to image
  qr_data TEXT,                          -- Raw QR string for regeneration
  
  -- Reference
  account_id UUID REFERENCES invoice_bank_details(id),
  
  -- Amount
  amount DECIMAL(12,2),                  -- Encoded in QR (if applicable)
  
  -- Status
  is_active BOOLEAN DEFAULT true,
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  
  INDEX idx_invoice (invoice_id),
  INDEX idx_type (qr_type),
  INDEX idx_account (account_id)
);
```

**Sample Data**:
```json
[
  {
    "id": "uuid",
    "business_id": "uuid",
    "invoice_id": "uuid",
    "qr_type": "upi",
    "qr_code_url": "https://s3.../upi-qr.png",
    "qr_data": "upi://pay?pa=abcshop@upi&pn=ABC&am=10000&tn=INV-001",
    "amount": 10000,
    "is_active": true
  },
  {
    "id": "uuid",
    "business_id": "uuid",
    "invoice_id": "uuid",
    "qr_type": "phone_pay",
    "qr_code_url": "https://s3.../phonepe-qr.png",
    "qr_data": "phonepe://pay?upiId=9876543210@phonepe&amount=10000",
    "amount": 10000,
    "is_active": true
  },
  {
    "id": "uuid",
    "business_id": "uuid",
    "invoice_id": "uuid",
    "qr_type": "bank_transfer",
    "qr_code_url": "https://s3.../bank-qr.png",
    "qr_data": "static-qr-image",
    "account_id": "uuid-of-bank-details",
    "amount": 10000,
    "is_active": true
  }
]
```

---

## 🔌 API Endpoints

### Purchase Orders

#### Create PO
```
POST /api/po
Authorization: Bearer {token}
X-Business-ID: {business_id}

{
  "po_number": "PO-2026-00123",
  "po_date": "2026-01-20",
  "supplier_id": "uuid",
  "delivery_date": "2026-02-20",
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
  "notes": "Urgent delivery required",
  "po_status": "draft"  // draft or issued
}

Response 201:
{
  "id": "uuid",
  "po_number": "PO-2026-00123",
  "status": "draft",
  "created_at": "2026-01-20T10:00:00Z"
}
```

#### List POs
```
GET /api/po?status=active&sort=-created_at&limit=20&offset=0
```

#### Update PO
```
PUT /api/po/:id
{
  "po_status": "issued",
  "delivery_date": "2026-02-20"
  // Can only update draft POs
}
```

#### Get PO Details
```
GET /api/po/:id

Response:
{
  "id": "uuid",
  "po_number": "PO-2026-00123",
  "supplier": {
    "id": "uuid",
    "name": "ABC Supplier",
    "email": "supplier@abc.com"
  },
  "items": [...],
  "total": 52500,
  "po_status": "accepted",
  "accepted_at": "2026-01-22T10:30:00Z"
}
```

#### Convert PO to Invoice
```
POST /api/po/:id/convert-to-invoice

{
  "invoice_date": "2026-02-20"
}

Response:
{
  "invoice_id": "uuid",
  "invoice_number": "INV-2026-00456"
}
```

#### Share PO
```
POST /api/po/:id/share/email

{
  "recipient_email": "supplier@abc.com",
  "message": "Please accept this PO"
}
```

---

### Bank Details Management

#### Add Bank Account
```
POST /api/business/bank-details
Authorization: Bearer {token}
X-Business-ID: {business_id}

{
  "account_holder_name": "ABC Enterprises",
  "account_number": "123456789012",
  "ifsc_code": "HDFC0001234",
  "bank_name": "HDFC Bank",
  "branch_name": "Pune",
  "upi_id": "abcshop@upi",
  "phone_pay_id": "9876543210@phonepe",
  "is_default": true
}

Response 201:
{
  "id": "uuid",
  "account_holder_name": "ABC Enterprises",
  "is_default": true
}
```

#### List Bank Accounts
```
GET /api/business/bank-details

Response:
{
  "data": [
    {
      "id": "uuid",
      "account_holder_name": "ABC Enterprises",
      "account_number": "****6789012",  // Masked
      "ifsc_code": "HDFC0001234",
      "is_default": true,
      "upi_qr_url": "https://s3.../upi-qr.png",
      "phone_pay_qr_url": "https://s3.../phonepe-qr.png"
    }
  ]
}
```

#### Generate QR Codes for Account
```
POST /api/business/bank-details/:id/generate-qr

Response:
{
  "upi_qr_url": "https://s3.../upi-qr.png",
  "phone_pay_qr_url": "https://s3.../phonepe-qr.png",
  "generated_at": "2026-01-20T10:00:00Z"
}
```

---

### Invoice Customization

#### Generate QR Codes for Invoice
```
POST /api/invoices/:id/generate-qr
Authorization: Bearer {token}

{
  "qr_types": ["upi", "phone_pay", "bank_transfer"],
  "amount": 10000,
  "bank_detail_id": "uuid"  // Optional, uses default if not provided
}

Response 201:
{
  "qr_codes": [
    {
      "qr_type": "upi",
      "qr_code_url": "https://s3.../upi-qr.png",
      "amount": 10000
    },
    {
      "qr_type": "phone_pay",
      "qr_code_url": "https://s3.../phonepe-qr.png",
      "amount": 10000
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

#### Get All QR Codes for Invoice
```
GET /api/invoices/:id/payment-qr

Response:
{
  "qr_codes": [...]
}
```

#### Download Invoice as PDF with QR Codes
```
GET /api/invoices/:id/as-pdf

Response: PDF file with:
- Logo/branding
- Invoice details
- All 3 QR codes (UPI, Phone Pay, Bank Transfer)
- Bank transfer details
- Payment terms
- Custom footer
```

---

## 🎨 Mobile UI Screens (Already Added to 08_UX.md)

### Owner Screens
1. **Create Purchase Order** (4-step flow)
2. **Bank Details Settings** (manage accounts + QR codes)
3. **Invoice Customization** (branding + payment options)
4. **PO Status Tracker** (monitor PO lifecycle)

### PDF Output
- Professional invoice with all 3 payment QR codes
- Bank transfer details clearly displayed
- Company logo and branding
- Payment terms and instructions

---

## 🔧 Technical Implementation

### Backend Services Required

**QR Code Generation Service**:
```typescript
interface QRCodeService {
  generateUPIQR(upiId: string, amount: number): Promise<string>;
  generatePhonePayQR(phonePayId: string, amount: number): Promise<string>;
  generateBankTransferQR(bankDetails: BankDetails): Promise<string>;
  saveQRToS3(qrData: Buffer, filename: string): Promise<string>;
}
```

**Invoice Customization Service**:
```typescript
interface InvoiceCustomizationService {
  addBankDetails(invoiceId: string, bankId: string): Promise<void>;
  addQRCodes(invoiceId: string, qrCodes: QRCode[]): Promise<void>;
  generatePDFWithQRs(invoiceId: string): Promise<Buffer>;
}
```

**Purchase Order Service**:
```typescript
interface POService {
  createPO(poData: POData): Promise<PO>;
  sendPOToSupplier(poId: string, email: string): Promise<void>;
  convertPOToInvoice(poId: string): Promise<Invoice>;
  trackPOStatus(poId: string): Promise<POStatus>;
}
```

### External API Integrations

**Razorpay QR API**:
- Generate UPI QR codes
- Generate dynamic QR with amount
- Fetch generated QR URLs

**Phone Pay QR API**:
- Generate Phone Pay QR codes
- Support for UPI ID based QR

**PDF Generation**:
- Use `pdfkit` or `puppeteer` for PDF creation
- Include images (logo, QR codes)
- Support multiple pages if needed

**AWS S3**:
- Upload QR code images
- Store with business_id prefix
- Set public read permissions

### Database Queries

**Find Bank Details for Business**:
```sql
SELECT * FROM invoice_bank_details 
WHERE business_id = $1 AND is_default = true AND is_active = true
LIMIT 1
```

**Get PO by Number**:
```sql
SELECT po.*, party.name AS supplier_name
FROM po_master po
JOIN parties party ON po.supplier_id = party.id
WHERE po.business_id = $1 AND po.po_number = $2
```

**Get Invoice QR Codes**:
```sql
SELECT * FROM payment_qr_codes
WHERE invoice_id = $1 AND is_active = true
ORDER BY created_at ASC
```

---

## 📊 Success Metrics

### Adoption Metrics
- % businesses with bank details configured: Target >60%
- % invoices with QR codes: Target >70%
- % POs created successfully: Target >50%
- Avg POs per business per month: Target >3

### Payment Metrics
- QR code scan rate on invoices: Target >25%
- Payment success rate via QR: Target >85%
- Avg time to payment after QR invoice: Target 3 days
- UPI vs Phone Pay vs Bank split: Monitor

### User Satisfaction
- NPS for PO feature: Target >40
- Invoice PDF rating: Target >4.5/5
- Support tickets for bank details: Target <2% of users

---

## ⚠️ Risks & Mitigation

### Risk 1: QR Code Generation Failures
**Impact**: Invoices without payment QR codes
**Mitigation**: 
- Fallback to static bank details QR
- Retry logic with exponential backoff
- Alert user if QR generation fails

### Risk 2: Bank Account Data Security
**Impact**: Sensitive account numbers exposed
**Mitigation**:
- Encrypt account numbers at rest
- Mask account numbers in UI (show last 4 digits)
- Rate limit bank detail API requests
- Audit all access to bank details

### Risk 3: PO Supplier Rejection
**Impact**: Suppliers reject POs, orders blocked
**Mitigation**:
- Pre-validation with supplier (check email)
- Retry email delivery if bounce
- Show delivery status to user
- Manual override option

### Risk 4: QR Code Scanning Issues
**Impact**: Customers can't scan QR codes
**Mitigation**:
- Generate high-resolution QR codes (500x500px minimum)
- Test with popular QR apps (Google Lens, etc.)
- Provide text-based payment link as fallback
- Clear instructions on invoice

---

## 📋 Implementation Checklist

### Phase 1: Database & Core API (Week 21)
- [x] Create 3 new tables (invoice_bank_details, po_master, payment_qr_codes)
- [ ] Write database migrations
- [ ] Add indexes for performance
- [ ] Add Row Level Security (RLS) policies

### Phase 2: API Endpoints (Week 22)
- [ ] Bank Details API (POST, GET, PUT, DELETE, generate-qr)
- [ ] Purchase Order API (POST, GET, PUT, convert-to-invoice, share)
- [ ] Invoice QR API (POST, GET generate-qr)
- [ ] PDF generation endpoint with QR codes
- [ ] Add input validation
- [ ] Add error handling

### Phase 3: QR Code Generation (Week 22-23)
- [ ] Integrate Razorpay QR API
- [ ] Integrate Phone Pay QR API
- [ ] Implement QR storage on S3
- [ ] Add retry logic for failed generations
- [ ] Create fallback for static QR

### Phase 4: Mobile UI (Week 23)
- [ ] PO creation flow (4 steps)
- [ ] Bank details screen
- [ ] Invoice customization screen
- [ ] PO status tracker
- [ ] Integration with existing invoice creation

### Phase 5: PDF Generation & Testing (Week 24)
- [ ] Update PDF generation to include all QR codes
- [ ] Add branding (logo, colors, footer)
- [ ] Test with various QR readers
- [ ] Load test QR generation
- [ ] User acceptance testing

### Phase 6: Documentation & Launch (Week 25)
- [ ] User documentation
- [ ] Admin guide for bank setup
- [ ] API documentation
- [ ] Supplier onboarding guide
- [ ] Launch to production

---

## 💾 Data Migration

**Existing Data Impact**: None
- New tables don't affect existing invoices
- Bank details are optional (backward compatible)
- POs are new feature (no migration needed)
- QR codes generated on-demand

**Migration Strategy**:
1. Create new tables in production
2. Run RLS policy setup
3. Add sample bank details for testing
4. Deploy API endpoints (feature-flagged)
5. Enable in mobile app gradually
6. Monitor for 2 weeks before full rollout

---

## 🚀 Launch Strategy

### Soft Launch (Week 25)
- Enable for 10% of businesses (beta users)
- Collect feedback on PO creation
- Monitor QR code generation success rate
- Check for data/security issues

### General Availability (Week 26)
- Enable for all businesses
- Send email tutorial: "Create Professional Purchase Orders"
- Add in-app onboarding for PO creation
- Monitor adoption metrics

### Phase 2 (Month 3+)
- Add supplier acceptance notifications (SMS/Email)
- Add auto-reminder for pending PO acceptance
- Add PO-to-Invoice conversion automation
- Add bulk PO creation from templates

---

## ✨ Related Features

- **Feature #21**: Invoice Templates (use for PO templates)
- **Feature #25**: Payment Portal (show QR codes from invoices)
- **Feature #29**: SMS/Email Notifications (notify suppliers of PO)
- **Feature #33**: Customer Discovery (auto-register suppliers)
- **Feature #15**: GST Compliance (include in PO documents)

---

## 📚 Documentation Links

- [Database Schema](01_ARCHITECTURE.md#tables-25-27)
- [API Specification](03_API.md#2b-purchase-orders--advanced-invoicing-api)
- [Mobile UX Flows](08_UX.md#purchase-orders--advanced-invoicing-screens)
- [Business Model Impact](05_BUSINESS.md)

---

**Last Updated**: January 2026  
**Status**: Ready for Implementation
