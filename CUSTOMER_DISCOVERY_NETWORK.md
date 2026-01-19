# Customer Discovery & Network Effect - Complete Specification

**Status**: CRITICAL FEATURE - Ready for Implementation  
**Priority**: P0 - Define Competitive Advantage  
**Phase**: Phase 4 (Parallel with Payment Portal)  
**Timeline**: 4-5 weeks (Weeks 12-16)

---

## 🎯 Executive Summary

This feature transforms SmartKhata from a standalone app into a **two-sided network** where:
1. **Owners** search for customers and send invoices
2. **Customers** auto-register, get notifications, and view their balance
3. **Network grows exponentially** (more customers = more stickiness)

**Business Impact**:
- 40% reduction in customer acquisition time (auto-registration)
- 3x faster payment collection (customers who get notification)
- Network effect (customers become repeat users)
- Competitive edge vs Khatabook + Tally (hybrid B2B/B2C platform)

---

## 📋 Problem Statement

### Owner Pain Points
1. **Manual Customer Creation**: Takes 5 minutes per new customer
2. **No Customer Visibility**: Customers don't know their balance
3. **Closed System**: Only owner has access to data
4. **Slow Collections**: Customers unaware of payment deadlines

### Customer Pain Points (Untapped Opportunity)
1. **No Invoice Visibility**: Customers don't get receipts/invoices
2. **Trust Issues**: No transparent record of transactions
3. **No Self-Service**: Can't view balance independently
4. **No Notification**: Doesn't know when invoice is ready

### Market Opportunity
- **Khatabook**: Customer-first approach (customers see their ledger)
- **Tally**: Seller-only (closed system)
- **SmartKhata Gap**: Need BOTH (open two-sided network)

---

## ✨ Solution Architecture

### 3-Layer Design

**Layer 1: Owner-Side (Search & Send)**
```
Owner searches "Ramesh" 
  → Found (registered customer)
  → OR Not found (create new)
     → Auto-register with phone
     → Send SMS with notification
```

**Layer 2: Customer-Side (Receive & View)**
```
Customer receives SMS/Email
  → Click link
  → Auto-verify OTP
  → View all invoices from all businesses
  → See outstanding balance
  → Pay online
```

**Layer 3: Network (Analytics & Growth)**
```
System tracks:
  → Customer engagement metrics
  → Payment velocity impact
  → Network health KPIs
  → Growth trends
```

---

## 🗄️ Database Schema

### Table 22: customer_notifications

Tracks all notifications sent to customers.

```sql
CREATE TABLE customer_notifications (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  customer_id UUID NOT NULL REFERENCES parties(id),
  invoice_id UUID NOT NULL REFERENCES invoices(id),
  
  notification_type VARCHAR(50),   -- invoice_created, payment_reminder, receipt_sent, welcome
  channel VARCHAR(20),             -- sms, email, push
  recipient VARCHAR(255),          -- Phone or email where sent
  
  subject TEXT,                    -- For email
  message_body TEXT,               -- Full message
  
  status VARCHAR(20) DEFAULT 'pending', -- pending, sent, failed, opened, bounced
  sent_at TIMESTAMP,
  opened_at TIMESTAMP,             -- For email tracking
  failure_reason TEXT,             -- If failed
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_customer_recent (customer_id, created_at DESC),
  INDEX idx_status_pending (status, created_at)
);
```

**Key Fields**:
- `notification_type`: Different flows (invoice, reminder, receipt, welcome)
- `channel`: SMS has delivery guarantee; Email has open tracking
- `status`: Granular tracking (pending→sent→opened)
- Indexes for: "Get all notifications for customer", "Find unsent messages"

---

### Table 23: customer_network_profile

Stores customer registration & engagement data.

```sql
CREATE TABLE customer_network_profile (
  id UUID PRIMARY KEY,
  customer_id UUID NOT NULL REFERENCES parties(id),
  
  -- Verification Status
  phone_verified BOOLEAN DEFAULT false,
  phone_verified_at TIMESTAMP,
  email_verified BOOLEAN DEFAULT false,
  email_verified_at TIMESTAMP,
  
  -- Profile Info
  profile_name VARCHAR(255),       -- Display name (may differ from invoice name)
  profile_type VARCHAR(50),        -- individual, business, farmer, retailer, corporate
  profile_photo_url TEXT,          -- S3 URL if uploaded
  
  -- Engagement Tracking
  first_invoice_date DATE,         -- When first invoice tagged to this customer
  last_login_date TIMESTAMP,       -- Last portal access
  total_invoices_received INT DEFAULT 0,
  total_amount_received DECIMAL(12,2) DEFAULT 0,
  total_payments_made DECIMAL(12,2) DEFAULT 0,
  
  -- Preferences
  portal_access_enabled BOOLEAN DEFAULT true,
  notifications_enabled BOOLEAN DEFAULT true,
  preferred_language VARCHAR(10) DEFAULT 'hi',
  preferred_channels VARCHAR(100),   -- "sms,email" or "sms_only"
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(customer_id)
);
```

**Key Fields**:
- `phone_verified`, `email_verified`: Two-step verification
- `profile_type`: Categorize for analytics (who uses system most)
- `first_invoice_date`: Cohort analysis
- `preferred_language`: Support multi-language SMS/Email (Hindi/English/Regional)
- Preferences: Customer controls notification frequency

---

### Table 24: customer_engagement_metrics

Daily snapshot of network health.

```sql
CREATE TABLE customer_engagement_metrics (
  id UUID PRIMARY KEY,
  business_id UUID NOT NULL REFERENCES businesses(id),
  
  metric_date DATE NOT NULL,
  
  -- Growth Metrics
  new_customers_count INT DEFAULT 0,
  verified_customers_count INT DEFAULT 0,
  active_customers_count INT DEFAULT 0,        -- Logged in last 7 days
  
  -- Engagement Metrics
  invoices_sent_count INT DEFAULT 0,
  notifications_sent_count INT DEFAULT 0,
  notification_open_rate DECIMAL(5,2),         -- % emails opened
  notification_delivery_rate DECIMAL(5,2),     -- % SMS delivered
  
  -- Payment Impact
  avg_payment_days DECIMAL(5,2),
  payment_velocity_impact DECIMAL(5,2),        -- % faster with notification
  collection_rate DECIMAL(5,2),                -- % of invoices paid
  
  -- Network Health
  portal_access_rate DECIMAL(5,2),             -- % of customers who logged in
  repeat_customer_rate DECIMAL(5,2),           -- % with 2+ invoices
  customer_satisfaction_score INT,              -- NPS-style tracking
  
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  UNIQUE(business_id, metric_date)
);
```

**Purpose**: 
- Daily rollup of all network metrics
- Owner dashboard visualization
- Identify trends (is network growing?)
- Compare before/after notification campaigns

---

## 🔌 API Endpoints (10 Total)

### 1. Customer Search

```http
GET /api/customers/search?q=Ramesh&limit=10
Authorization: Bearer {access_token}
X-Business-ID: {business_id}
```

**Response**:
```json
{
  "results": [
    {
      "type": "exact_match",
      "customer_id": "uuid-123",
      "name": "Ramesh Kumar",
      "phone": "+919876543210",
      "profile_type": "individual",
      "status": "registered",
      "phone_verified": true,
      "last_invoice": "2025-01-15",
      "total_balance": 5000,
      "portal_active": true
    },
    {
      "type": "suggestion",
      "customer_id": "NEW",
      "name": "Ramesh Sharma",
      "phone": null,
      "profile_type": null,
      "status": "not_registered",
      "action": "create_new"
    }
  ],
  "search_took_ms": 45
}
```

**Logic**:
- Search by name (partial match) → phone → email
- Return exact matches first
- Suggest similar names (Levenshtein distance)
- One "CREATE NEW" option at end

---

### 2. Auto-Register Customer

```http
POST /api/customers/auto-register
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

{
  "name": "Ramesh Kumar",
  "phone": "+919876543210",
  "email": "ramesh@email.com",
  "profile_type": "individual",
  "auto_send_welcome_sms": true,
  "auto_tag_recent_invoices": false
}
```

**Response**:
```json
{
  "customer_id": "uuid-456",
  "name": "Ramesh Kumar",
  "phone_verified": false,
  "email_verified": false,
  "otp_sent_to_phone": true,
  "otp_resend_after": 60,
  "message": "Customer created. OTP sent to +919876543210"
}
```

**Flow**:
1. Create entry in `parties` table (links to business)
2. Create entry in `customer_network_profile`
3. Send OTP via SMS (MSG91 API)
4. Return customer_id for immediate use
5. Owner can send invoice right away (customer gets notified when they verify)

---

### 3. Send Invoice to Customer

```http
POST /api/invoices/{invoice_id}/send-to-customer
Authorization: Bearer {access_token}
X-Business-ID: {business_id}

{
  "customer_id": "uuid-456",
  "channels": ["sms", "email"],
  "message_template": "default",    -- or "urgent", "reminder"
  "custom_message": null,
  "include_payment_link": true,
  "include_pdf_attachment": true
}
```

**Response**:
```json
{
  "notification_id": "uuid-789",
  "invoice_id": "uuid-456",
  "customer_id": "uuid-456",
  "channels_sent": ["sms", "email"],
  "sms": {
    "status": "sent",
    "message_id": "msg-001",
    "recipient": "+919876543210"
  },
  "email": {
    "status": "sent",
    "message_id": "email-001",
    "recipient": "ramesh@email.com"
  },
  "created_at": "2025-01-20T10:30:00Z"
}
```

**Behavior**:
- Create entry in `customer_notifications` table
- Generate SMS via MSG91 (immediate delivery)
- Generate Email via SendGrid (with open tracking)
- Include payment link from `payment_links` table
- Attach PDF if requested
- Track delivery status in real-time

---

### 4. Get Customer Network Analytics

```http
GET /api/analytics/customer-network
Authorization: Bearer {access_token}
X-Business-ID: {business_id}
```

**Response**:
```json
{
  "period": "last_30_days",
  "summary": {
    "total_customers": 1250,
    "verified_customers": 980,
    "active_customers_7d": 450,
    "total_invoices_sent": 3500,
    "total_notifications": 3800
  },
  "engagement": {
    "notification_open_rate": 68,
    "notification_delivery_rate": 95,
    "portal_access_rate": 36,
    "repeat_customer_rate": 72
  },
  "payment_impact": {
    "avg_payment_days": 5,
    "avg_payment_days_pre_notification": 8,
    "payment_velocity_improvement": 35,
    "collection_rate": 87
  },
  "daily_trend": [
    {
      "date": "2025-01-20",
      "new_customers": 12,
      "verified": 10,
      "invoices_sent": 120,
      "notifications_sent": 125,
      "portal_logins": 45,
      "payments_received": 8500
    }
  ]
}
```

**Use Case**: Owner dashboard shows "network health"

---

### 5. Customer Portal: Register

```http
POST /api/customer-portal/register
(No Auth - Public)

{
  "phone": "+919876543210",
  "otp": "123456"
}
```

**Response**:
```json
{
  "customer_id": "uuid-456",
  "customer_name": "Ramesh Kumar",
  "token": "eyJhbGc...",
  "refresh_token": "eyJhbGc...",
  "businesses": [
    {
      "business_id": "uuid-111",
      "business_name": "Fertilizer Shop",
      "total_invoices": 4,
      "total_outstanding": 2500,
      "total_paid": 5000
    }
  ]
}
```

**Flow**:
1. Customer receives SMS with OTP
2. Customer enters OTP here
3. System verifies against `customer_notifications` table
4. Mark `phone_verified = true`
5. Generate JWT token
6. Return list of businesses (from invoices sent to this customer)

---

### 6. Customer Portal: View Invoices

```http
GET /api/customer-portal/invoices?business_id=uuid-111&status=unpaid
Authorization: Bearer {customer_token}
```

**Response**:
```json
{
  "business_name": "Fertilizer Shop",
  "invoices": [
    {
      "invoice_id": "uuid-555",
      "invoice_number": "INV-001",
      "date": "2025-01-20",
      "amount": 2500,
      "status": "unpaid",
      "due_date": "2025-02-05",
      "days_until_due": 16,
      "payment_link": "https://smartkhata.com/pay/uuid-555",
      "pdf_url": "https://s3.../INV-001.pdf"
    }
  ],
  "summary": {
    "total_invoices": 4,
    "total_outstanding": 2500,
    "total_paid": 5000
  }
}
```

---

### 7. Customer Portal: View Balance

```http
GET /api/customer-portal/balance?business_id=uuid-111
Authorization: Bearer {customer_token}
```

**Response**:
```json
{
  "business_name": "Fertilizer Shop",
  "total_outstanding": 2500,
  "total_paid_lifetime": 5000,
  "total_invoices": 4,
  "avg_invoice_amount": 1875,
  "last_transaction": "2025-01-20",
  "payment_methods": ["upi", "card", "netbanking"],
  "payment_link": "https://smartkhata.com/pay/business-uuid-111"
}
```

---

### 8. Notification Delivery Report

```http
GET /api/notifications/delivery-report?date_from=2025-01-20&date_to=2025-01-21
Authorization: Bearer {access_token}
X-Business-ID: {business_id}
```

**Response**:
```json
{
  "period": "2025-01-20 to 2025-01-21",
  "summary": {
    "total_sent": 150,
    "successful": 148,
    "failed": 2,
    "bounce_rate": 1.3
  },
  "by_channel": {
    "sms": {
      "sent": 90,
      "delivered": 90,
      "failed": 0,
      "delivery_rate": 100
    },
    "email": {
      "sent": 60,
      "delivered": 58,
      "failed": 2,
      "bounce_rate": 3.3,
      "open_rate": 68,
      "click_rate": 45
    }
  },
  "by_type": {
    "invoice_created": { "sent": 80, "open_rate": 70 },
    "payment_reminder": { "sent": 50, "open_rate": 65 },
    "receipt_sent": { "sent": 20, "open_rate": 75 }
  }
}
```

---

## 📱 Mobile UX Flows

### Owner Flow 1: Quick Invoice + Send

```
1. Create Invoice Screen
   └─ [Select Party]
      └─ Search field: "Ramesh"
         └─ Exact match found OR Create new
            └─ Select → Continue to items
2. Add Items Screen
   └─ [Add items...]
      └─ [Generate Invoice]
         └─ Confirm
3. Send Screen
   ├─ ☑ Send SMS
   ├─ ☑ Send Email
   └─ [Send Now]
      └─ "Invoice sent! OTP sent to Ramesh"
```

**Benefits**: 
- 0 extra steps for registered customers
- 1 extra step for new customers (auto-register)
- Total time: 3-5 minutes per invoice

---

### Owner Flow 2: Network Analytics View

```
1. Dashboard
   └─ [Customer Network Card]
      ├─ Total: 1,250
      ├─ Verified: 980 (78%)
      ├─ Active: 450 (36%)
      └─ [Tap to expand]
2. Network Details Screen
   ├─ Summary cards (growth, engagement, payment impact)
   ├─ Daily trend graph (30-day chart)
   ├─ Benchmark comparison (vs industry avg)
   └─ [Export Report]
```

---

### Customer Flow: First Time Login

```
1. SMS Received
   "Your invoice from Fertilizer Shop is ready. Pay: smartkhata.com/pay/xyz"
2. Customer clicks link
   └─ Smart Khata app opens (or web)
3. Phone Verification Screen
   └─ "+919876543210"
   └─ [Verify with OTP]
      └─ "Enter 6-digit OTP sent to +919876543210"
4. OTP Entered
   └─ [Verify]
      └─ Auto-login
5. Dashboard
   └─ "You have 1 unpaid invoice"
   ├─ Show outstanding
   ├─ Show recent transactions
   └─ [Pay Now]
```

---

### Customer Flow: Repeat Visits

```
1. Customer opens SmartKhata
   └─ Auto-logged in (remember token)
2. Dashboard
   └─ Shows:
      ├─ Businesses (Fertilizer Shop, Book Shop, Fuel Station)
      ├─ Total outstanding across all
      ├─ Recent invoices
      └─ Quick pay buttons
3. Select business
   └─ See all invoices from that business
   └─ Click invoice → [Pay] or [Download PDF]
```

---

## 📊 Success Metrics

### Phase 1 (Weeks 1-2): Implementation
- ✅ All 10 API endpoints deployed
- ✅ Customer notifications system working
- ✅ Portal auto-login working
- ✅ Analytics dashboard working

### Phase 2 (Weeks 3-4): Adoption
- **Customer Registration Rate**: > 80% complete verification within 7 days
- **Portal Login Rate**: > 50% of registered customers login within 30 days
- **Invoice Delivery**: > 95% SMS delivery, > 80% email delivery

### Phase 3 (Weeks 5+): Impact
- **Payment Velocity**: 35% faster (5 days vs 8 days)
- **Collection Rate**: 87% of invoices paid within 30 days
- **Network Growth**: 3x increase in customer engagement
- **Repeat Customers**: 72% have 2+ invoices (stickiness)

### Long-term (6+ months)
- **Network Expansion**: Exponential growth (word-of-mouth from customers)
- **Two-Sided Adoption**: Customers proactively checking portal
- **Competitive Advantage**: Become platform, not just app
- **Revenue**: Premium features for customers (analytics, bulk payment, etc.)

---

## 🛠️ Implementation Checklist

### Backend (Weeks 1-2)
- [ ] Create 3 new database tables (notifications, network_profile, metrics)
- [ ] Implement 10 API endpoints (search, register, send, analytics, portal)
- [ ] Setup OTP generation/verification (MSG91 integration)
- [ ] Setup email delivery with open tracking (SendGrid)
- [ ] Implement background job for daily metrics rollup
- [ ] Create customer_notifications worker (send queued messages)
- [ ] Add Row-Level Security for customer data (RLS policies)
- [ ] Write unit tests for search logic, auto-register, notification sending

### Mobile (Weeks 2-3)
- [ ] Implement "Search Customer" UI with autocomplete
- [ ] Implement "Auto-Register Customer" flow
- [ ] Add "Send to Customer" button on invoice detail screen
- [ ] Build customer portal login (OTP verification)
- [ ] Build customer portal dashboard (invoices + balance)
- [ ] Add "Network Analytics" screen to owner dashboard
- [ ] Implement notification badge/prompt ("New customer registered!")
- [ ] Add analytics export (PDF report)

### Testing (Week 4)
- [ ] End-to-end: Owner creates customer → Customer gets SMS → Customer logs in → Sees invoice
- [ ] Load test: 1000 concurrent notifications
- [ ] SMS delivery: 95%+ delivery rate
- [ ] Email open rate: Track via SendGrid webhooks
- [ ] Customer portal security: RLS verified
- [ ] Analytics accuracy: Daily metrics vs real data

### DevOps
- [ ] Setup MSG91 account (SMS provider)
- [ ] Setup SendGrid account (Email + tracking)
- [ ] Create background job scheduler (for daily metrics)
- [ ] Add monitoring for notification delivery
- [ ] Create dashboard for notification health metrics

---

## 🎯 Business Strategy

### Why This Feature Matters

1. **Solves Owner Problem**: Save 30+ hours/month on manual customer creation
2. **Solves Customer Problem**: Transparency (they get invoices too)
3. **Competitive Differentiation**: 
   - Khatabook = Customer-first (good, but seller-weak)
   - Tally = Seller-first (good, but no customer engagement)
   - SmartKhata = BOTH (hybrid platform)
4. **Network Effect**: Each new customer adds value to all existing customers
5. **Revenue Opportunity**: Premium features for customers (analytics, bulk payment, etc.)

### Go-to-Market

**Phase 1**: Launch with payment portal (week 12)
- "Now your customers can pay online + see their balance"
- "Stop sending invoices manually"

**Phase 2**: Add network analytics (week 16)
- "See how many customers are engaged"
- "Watch payment velocity improve"

**Phase 3**: Premium tier (month 3)
- "Customer portal analytics"
- "Multi-business tracking for customers"
- "Expense splitting between customers"

---

## 📞 Support & Documentation

### For Owners
- Tutorial: "How to search and tag customers"
- Tutorial: "Understanding network health metrics"
- FAQ: "Why did my customer not receive SMS?"
- FAQ: "How do I re-send a notification?"

### For Customers
- "How to login to your portal"
- "Understanding your invoice"
- "How to pay online"
- "Download your transaction history"

---

## ⚠️ Technical Risks & Mitigations

| Risk | Impact | Mitigation |
|------|--------|-----------|
| SMS delivery failures | Customer doesn't get invoice | Fallback to email, retry logic, owner alert |
| OTP expiry issues | Customer can't login | Extended OTP window (10 min), resend button |
| Phone number conflicts | Wrong customer gets SMS | Verification: match phone + first name |
| RLS security breach | Data leakage between businesses | Test RLS policies, audit logs |
| Notification spam | Customer disables notifications | Preference center, frequency capping |
| Portal doesn't scale | Performance issues at 100k customers | Index on customer_id, pagination, caching |

---

## 📄 References

- **Database Schema**: [01_ARCHITECTURE.md](01_ARCHITECTURE.md) - Tables 22-24
- **API Specs**: [03_API.md](03_API.md) - Customer Discovery & Network section
- **UX Flows**: [08_UX.md](08_UX.md) - Customer Discovery & Network Screens
- **Feature List**: [02_FEATURES.md](02_FEATURES.md) - Feature 33
- **Integration**: [09_INTEGRATIONS.md](09_INTEGRATIONS.md) - MSG91, SendGrid setup
