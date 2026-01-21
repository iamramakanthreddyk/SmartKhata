# 9. SmartKhata - Third-Party Integrations

**Purpose**: External API integrations (GST, payments, SMS, email, storage)  
**Audience**: Backend developers, integration engineers  
**Consolidates**: INTEGRATION_SPECS.md

---

## 🔌 Integration Architecture

**Pattern**: Queued Processing
```
SmartKhata API
  ↓
Job Queue (Redis)
  ↓
Integration Service
  ├─ GST Portal Service
  ├─ Payment Service
  ├─ SMS Service
  ├─ Email Service
  └─ Storage Service
  ↓
External APIs
```

**Error Handling**:
- Retry policy: 3 attempts with exponential backoff
- Webhook for result notifications
- Dead letter queue for failures

---

## 💰 Razorpay Integration

**Purpose**: Process payments for premium subscriptions

**Implementation**:
```javascript
// Create subscription
POST /razorpay/subscriptions
{
  "business_id": "biz_123",
  "plan_id": "plan_diamond",
  "customer_notify": 1
}

// Webhook listener
POST /webhooks/razorpay
{
  "event": "subscription.activated",
  "payload": { "subscription": { "id": "sub_123" } }
}
```

**API Credentials**:
- Key ID: Environment variable `RAZORPAY_KEY_ID`
- Key Secret: Environment variable `RAZORPAY_KEY_SECRET`

**Webhook Security**:
- Verify signature using HMAC-SHA256
- `X-Razorpay-Signature` header validation

---

## 📧 SendGrid Integration

**Purpose**: Send invoice emails, welcome emails, reminders

**Implementation**:
```javascript
// Send invoice email
POST /emails/send
{
  "to": "customer@example.com",
  "template_id": "sendgrid_template_id",
  "variables": {
    "invoice_number": "INV-001",
    "total_amount": "₹5000"
  }
}
```

**Email Templates**:
- Welcome email (onboarding)
- Invoice email (with PDF)
- Payment reminder
- Overdue invoice reminder

**Credentials**:
- API Key: `SENDGRID_API_KEY`

---

## 📱 MSG91 / WhatsApp Integration

**Purpose**: Send invoices and reminders via SMS/WhatsApp

**Implementation**:
```javascript
// Send WhatsApp invoice
POST /messages/whatsapp
{
  "phone": "919876543210",
  "template": "invoice_share",
  "variables": {
    "customer_name": "ABC Store",
    "invoice_link": "https://smartkhata.app/inv/123"
  }
}

// Send SMS reminder
POST /messages/sms
{
  "phone": "919876543210",
  "text": "Invoice INV-001 for ₹5000 is pending. Reply to pay: [link]"
}
```

**Credentials**:
- Auth Key: `MSG91_AUTH_KEY`
- Route: `transactional`

**Message Templates** (pre-approved):
- Invoice SMS reminder
- Invoice WhatsApp template
- Payment received confirmation
- Invoice overdue reminder

---

## 🧾 GST Portal Integration

**Purpose**: Auto-submit GSTR-1, GSTR-2, generate IRN for e-invoicing

### GST Rate Configuration
**How rates are determined**:
- SmartKhata maintains HSN-to-GST-Rate master table (updated annually)
- Standard GST rates: 0%, 5%, 12%, 18%, 28%
- For e-invoicing, rates are sent as single value but split as follows:
  - **Intra-state**: Calculated as CGST + SGST in GSTR filings
  - **Inter-state**: Sent as IGST to GST portal
- User can override rate if:
  - HSN not in master table (manual entry required)
  - Exemption or special rate applies
- Rate validation: Only allows standard GST rates (0%, 5%, 12%, 18%, 28%)

### E-Invoicing (IRN Generation)

**API Endpoint** (via Razorpay Route):
```
POST https://api.gst.razorpay.in/invoices
```

**Request**:
```json
{
  "businessGSTIN": "06AABCT1234H1Z0",
  "invoiceNumber": "INV-2026-001",
  "invoiceDate": "2026-01-19",
  "invoiceType": "B2B",
  "documentType": "INV",
  "customerGSTIN": "08AABCT1234H1Z0",
  "customerState": "KA",
  "businessState": "KA",
  "lineItems": [
    {
      "itemName": "Item 1",
      "hsnCode": "271121",
      "itemQuantity": 10,
      "itemPrice": 100,
      "itemTaxRate": 18,
      "cgstRate": 9,
      "sgstRate": 9
    }
  ],
  "totalTaxableValue": 1000,
  "totalCGSTAmount": 90,
  "totalSGSTAmount": 90,
  "totalTaxAmount": 180,
  "totalInvoiceAmount": 1180
}
```

**State Logic**:
- If `customerState == businessState`: Send CGST + SGST (split equally)
- If `customerState != businessState`: Send IGST (18% becomes 18% IGST, not split)

**Response**:
```json
{
  "irn": "XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX",
  "ackNo": "XXXXXXXXXXX",
  "ackDt": "2026-01-19T10:30:00"
}
```

**IRN Validity**:
- 1 IRN per invoice (immutable)
- Store with invoice record
- Include in PDF (as QR code)

### GSTR-1 Auto-Submission

**Monthly Process**:
1. Collect all B2B invoices from month
2. Calculate taxable values by GSTIN
3. Group by CGST/SGST/IGST based on party state
4. Generate GSTR-1 JSON
5. Submit to GST portal
6. Capture ACK number

**Rate Logic in GSTR-1**:
- For each line item, send HSN code + tax rate
- If intra-state: Show CGST + SGST separately
- If inter-state: Show IGST amount

**Implementation**:
```javascript
// Auto-generate GSTR-1
POST /gst/gstr1/generate
{
  "month": "01",
  "year": "2026"
}

// Submit to GST portal
POST /gst/gstr1/submit
{
  "gstr1_id": "gstr1_123"
}
```

---

## 📋 HSN/SAC Master Table

**What SmartKhata Maintains**:
```
HSN Code  Description                  GST Rate  Category
─────────────────────────────────────────────────────────
0201      Beef                          5%        Meat
1211      Medicinal plants              0%        Pharma
2713      Petroleum coke                0%        Fuel
6204      Garments                      12%       Textiles
8504      Electrical machinery          18%       Electrical
9302      Weapons                       28%        Sin goods
```

**Rate Lookup Process**:
1. User enters HSN code on item
2. SmartKhata queries master table
3. If found: Auto-applies rate to item
4. If not found: User manually enters rate
5. User can override if rate changed
6. Rate locked per invoice line (audit trail)

**Manual Override Validation**:
- Only standard rates allowed: 0%, 5%, 12%, 18%, 28%
- No custom rates (7%, 15%, 20% rejected)
- Reason logged in audit trail for compliance

---

### GSTR-1 Auto-Submission

**Monthly Process**:
1. Collect all B2B invoices from month
2. Calculate taxable values by GSTIN
3. Generate GSTR-1 JSON
4. Submit to GST portal
5. Capture ACK number

**Implementation**:
```javascript
// Auto-generate GSTR-1
POST /gst/gstr1/generate
{
  "month": "01",
  "year": "2026"
}

// Submit to GST portal
POST /gst/gstr1/submit
{
  "gstr1_id": "gstr1_123"
}
```

---

## 🖼️ AWS Textract (OCR)

**Purpose**: Extract data from bill photos using OCR

**Implementation**:
```javascript
// Analyze bill image
POST /ocr/analyze
{
  "image_url": "https://s3.amazonaws.com/bills/bill123.jpg"
}

// Response
{
  "vendor_name": "ABC Suppliers",
  "invoice_number": "2026-001",
  "invoice_date": "2026-01-18",
  "amount": 5000,
  "tax": 900,
  "total": 5900,
  "confidence": 0.95
}
```

**Process**:
1. User uploads bill photo
2. Send to AWS Textract
3. Extract text and tables
4. Parse with NLP
5. Pre-fill purchase entry form
6. User verifies and saves

**S3 Bucket**: `smartkhata-bills/`
- Retention: 30 days
- Encryption: AES-256
- Access: Private, VPC endpoint only

---

## 💾 AWS S3 Integration

**Purpose**: Store PDF invoices, bills, receipts

**Bucket Structure**:
```
smartkhata-prod/
├── invoices/
│   └── business_id/
│       ├── INV-001.pdf
│       ├── INV-002.pdf
├── bills/
│   └── business_id/
│       └── BILL-001.jpg
└── receipts/
    └── business_id/
        └── RECV-001.pdf
```

**CDN**: CloudFront distribution for fast delivery

**Access**:
- Pre-signed URLs (valid 24 hours)
- Direct S3 API calls via IAM role
- Public URLs for shared invoices

---

## 📊 Mixpanel Integration

**Purpose**: Product analytics and user behavior tracking

**Events Tracked**:
```javascript
// Onboarding events
- "user_signup"
- "business_created"
- "first_invoice_created"

// Feature usage
- "invoice_created"
- "invoice_shared"
- "invoice_paid"
- "gst_report_viewed"

// Payment events
- "subscription_started"
- "subscription_cancelled"
- "payment_failed"
```

**Implementation**:
```javascript
import Mixpanel from 'mixpanel-browser';

Mixpanel.track('invoice_created', {
  amount: 5000,
  tax_rate: 18,
  shared_via: 'whatsapp'
});
```

---

## 🔐 API Credential Management

**Store credentials**:
- `RAZORPAY_KEY_ID` → AWS Secrets Manager
- `RAZORPAY_KEY_SECRET` → AWS Secrets Manager
- `SENDGRID_API_KEY` → AWS Secrets Manager
- `MSG91_AUTH_KEY` → AWS Secrets Manager
- `AWS_TEXTRACT_KEY` → IAM role
- `MIXPANEL_TOKEN` → Application config

**Rotation Policy**:
- Every 90 days for API keys
- Every 180 days for access keys
- Immediately if compromised

---

## ⚠️ Error Handling

**Rate Limiting** (per service):
- Razorpay: 100 req/second
- SendGrid: 300 req/second
- MSG91: 50 SMS/second
- GST Portal: 10 req/second

**Retry Strategy**:
```
Attempt 1: Immediate
Attempt 2: After 5 seconds
Attempt 3: After 30 seconds
After 3 failures: Manual intervention
```

---

## 📋 Integration Checklist

- [ ] Razorpay account created & live keys obtained
- [ ] SendGrid account created & templates configured
- [ ] MSG91 account created & templates approved
- [ ] GST Portal registered for e-invoicing
- [ ] AWS IAM role created for S3 & Textract
- [ ] Mixpanel project created
- [ ] All credentials in AWS Secrets Manager
- [ ] Webhook endpoints configured
- [ ] Error handling & logging implemented
- [ ] Rate limiting configured

---

**Related Documents**: 03_API.md, 04_SECURITY.md, 06_OPERATIONS.md, 07_DEVELOPMENT.md
