# 4. SmartKhata - Security & Compliance

**Purpose**: Security architecture, RLS, audit trails, auth, compliance  
**Audience**: DevOps, security, backend developers  
**Consolidates**: SECURITY_POLICY.md, RBAC_FEATURE_CONTROL_ANALYSIS.md, COMPLIANCE_FRAMEWORK.md, IP_PROTECTION.md

---

## 🔐 Security Architecture

### Defense-in-Depth Strategy

**Layer 1: Database** (Row-Level Security)
- RLS policies prevent cross-business data access
- Every query automatically scoped to business_id

**Layer 2: API** (Permission Middleware)
- Every endpoint checks user permissions
- Consistent permission matrix enforced
- Feature control checked on access

**Layer 3: Application** (Service Layer)
- Business logic validates all operations
- No direct database access from API

**Layer 4: Audit** (Logging)
- All access attempts logged
- Permission changes tracked
- Compliance audit trail maintained

---

## 🔑 Authentication & Authorization

### Authentication (JWT)

**Process**:
1. User logs in with email/password
2. System validates credentials (bcrypt)
3. Issues JWT tokens:
   - `access_token` (15 minutes)
   - `refresh_token` (7 days)

**JWT Payload**:
```json
{
  "user_id": "uuid",
  "business_id": "uuid",
  "email": "user@example.com",
  "role": "owner|manager|staff",
  "permissions": ["invoice.create", "invoice.view", ...],
  "iat": 1642620000,
  "exp": 1642620900
}
```

**Refresh Token Flow**:
```
1. access_token expires
2. Client sends refresh_token to /auth/refresh
3. System validates refresh_token
4. Issues new access_token
```

---

### Authorization (RBAC + Features + Objects)

**3-Level Permission Check**:

**Level 1: Role-Based**:
- `owner` - Full access
- `manager` - Most features, no settings
- `staff` - Limited to assigned tasks

**Level 2: Feature-Based**:
- Feature disabled? → No access
- Feature limit exceeded? → Error

**Level 3: Object-Level**:
- Staff sees only own invoices
- Staff sees assigned parties only
- Managers see all business data

**Middleware Chain** (on every request):
```typescript
middleware(req, res, next) {
  // 1. Verify JWT token
  const user = verifyJWT(req.headers.authorization);
  
  // 2. Check role permission
  if (!hasRolePermission(user.role, req.endpoint)) {
    return res.status(403).send("Insufficient role");
  }
  
  // 3. Check feature enabled
  const feature = getFeatureFor(req.endpoint);
  if (!isFeatureEnabled(user.business_id, feature)) {
    return res.status(403).send("Feature not enabled");
  }
  
  // 4. Set database context (for RLS)
  setDatabaseContext(user.user_id, user.business_id);
  
  next();
}
```

---

## 🏛️ Row-Level Security (RLS)

### PostgreSQL RLS Policies

**Every table follows this pattern**:

```sql
-- Enable RLS
ALTER TABLE invoices ENABLE ROW LEVEL SECURITY;

-- Policy: Users can only see invoices from their business
CREATE POLICY business_isolation ON invoices
  USING (business_id = current_business_id());

-- Policy: Only invoice creator or manager can UPDATE
CREATE POLICY invoice_update ON invoices
  FOR UPDATE
  USING (
    business_id = current_business_id() AND
    (created_by = current_user_id() OR current_role() = 'manager')
  );
```

**Result**: Even if a malicious user bypasses API permission checks, the database RLS prevents data leakage.

---

### Feature Control at Database Level

**Feature Flags Table**:
```sql
CREATE TABLE feature_flags (
  id UUID PRIMARY KEY,
  business_id UUID REFERENCES businesses(id),
  feature_name VARCHAR(255),  -- 'invoicing', 'e-invoicing', etc.
  is_enabled BOOLEAN DEFAULT false,
  usage_count INT DEFAULT 0,
  usage_limit INT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**Database Function for Feature Checking**:
```sql
CREATE OR REPLACE FUNCTION is_feature_enabled(
  p_business_id UUID,
  p_feature_name VARCHAR
) RETURNS BOOLEAN AS $$
BEGIN
  RETURN EXISTS(
    SELECT 1 FROM feature_flags
    WHERE business_id = p_business_id
      AND feature_name = p_feature_name
      AND is_enabled = true
      AND (usage_limit IS NULL OR usage_count < usage_limit)
  );
END;
$$ LANGUAGE plpgsql;
```

---

## 📝 Audit Trail

### Audit Logging

**Every database change is logged**:
```sql
CREATE TABLE audit_logs (
  id UUID PRIMARY KEY,
  business_id UUID REFERENCES businesses(id),
  staff_id UUID REFERENCES staff(id),
  table_name VARCHAR(100),
  record_id UUID,
  action VARCHAR(50), -- INSERT, UPDATE, DELETE
  old_values JSONB,   -- Previous state
  new_values JSONB,   -- New state
  ip_address VARCHAR(45),
  user_agent TEXT,
  timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  INDEX idx_audit_search (table_name, record_id, timestamp)
);
```

**Trigger on Every Table**:
```sql
CREATE OR REPLACE FUNCTION audit_trigger_function() RETURNS TRIGGER AS $$
BEGIN
  IF TG_OP = 'INSERT' THEN
    INSERT INTO audit_logs(business_id, staff_id, table_name, record_id, action, new_values)
    VALUES(current_business_id(), current_user_id(), TG_TABLE_NAME, NEW.id, 'INSERT', row_to_json(NEW));
  ELSIF TG_OP = 'UPDATE' THEN
    INSERT INTO audit_logs(business_id, staff_id, table_name, record_id, action, old_values, new_values)
    VALUES(current_business_id(), current_user_id(), TG_TABLE_NAME, NEW.id, 'UPDATE', row_to_json(OLD), row_to_json(NEW));
  END IF;
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- Apply to all tables
CREATE TRIGGER audit_trigger AFTER INSERT OR UPDATE ON invoices FOR EACH ROW EXECUTE audit_trigger_function();
```

**Audit Queries** (for investigations):
```sql
-- Who changed this invoice?
SELECT * FROM audit_logs WHERE table_name = 'invoices' AND record_id = 'xyz' ORDER BY timestamp DESC;

-- What did this staff member do today?
SELECT * FROM audit_logs WHERE staff_id = 'xyz' AND DATE(timestamp) = CURRENT_DATE;

-- All changes to a party
SELECT * FROM audit_logs WHERE table_name = 'parties' AND record_id = 'xyz';
```

---

## 🔒 Data Protection

### Encryption at Rest

**Sensitive Fields** (encrypted):
- passwords (bcrypt hash, not reversible)
- GST numbers (AES-256)
- Bank account info (AES-256)

**Implementation**:
```typescript
import crypto from 'crypto';

function encryptSensitive(plaintext: string): string {
  const cipher = crypto.createCipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
  let encrypted = cipher.update(plaintext, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}

function decryptSensitive(encrypted: string): string {
  const decipher = crypto.createDecipher('aes-256-cbc', process.env.ENCRYPTION_KEY);
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  return decrypted;
}
```

### Encryption in Transit

**HTTPS Only**:
- All API calls via HTTPS (TLS 1.3)
- No HTTP fallback
- HSTS header enforced

---

## 🔄 Data Access Controls

### Object-Level Permissions

**Staff Can Only See**:
- Invoices they created
- Parties assigned to them
- Their own payment records

**Managers Can See**:
- All invoices from business
- All parties
- All payments

**Implementation** (in repositories):
```typescript
// For staff user, filter by created_by
if (user.role === 'staff') {
  query.where({ created_by: user.id });
}

// For manager or owner, no filter
```

---

## 📋 Compliance Requirements

### GDPR Compliance

**Data Collection**:
- Only collect necessary data (email, business info)
- Users can request data export
- Data deletion (right to be forgotten)

**Data Processing**:
- Privacy policy available
- Consent recorded for email marketing
- Third-party integrations disclosed

**API Endpoints** (for GDPR):
```
GET    /api/privacy/my-data             - Export all user data
POST   /api/privacy/delete-account      - Request account deletion
```

### GST Compliance (India)

**Requirements**:
- GST Number validation
- E-Invoicing for B2B transactions
- GSTR-1 monthly filing
- HSN/SAC codes for items

**Automatic Validation**:
- GST number format (15 chars, alphanumeric)
- E-Invoicing generates IRN on GSTN
- GSTR-1 preparation via Reports API

### IT Act 2000 (India)

**Requirements**:
- Digital signatures on invoices
- Audit trail maintenance
- Data retention (7 years for financial)

**Implementation**:
- QR code + digital signature on e-invoices
- Audit logs maintained for 7+ years
- No automatic purging of records

---

## 🛡️ Intellectual Property Protection

### Software & Code

- **License**: Proprietary (not open source)
- **Source Code Access**: Restricted to team
- **Employee IP Agreements**: All code created belongs to company

### Trademarks

- **Registered**: SmartKhata®
- **Usage**: Only company can use in product/marketing

### Patents

- **Candidates**: AI reconciliation, OCR integration
- **Filing**: Via patent attorneys

### Trade Secrets

- **Classification**: Business logic, algorithms, pricing model
- **Protection**: NDA with employees and partners

---

## 📞 Incident Response

### Security Incident Steps

1. **Detect** - Log review, alerts, user reports
2. **Contain** - Disable affected access, halt data access
3. **Investigate** - Audit logs, determine scope
4. **Notify** - Affected users within 24 hours
5. **Remediate** - Fix vulnerability, patch system
6. **Review** - Post-incident analysis, prevent recurrence

### Contacts

- **Security Lead**: contact@example.com
- **Legal**: legal@example.com
- **Escalation**: board@example.com

---

**Related Documents**: 01_ARCHITECTURE.md, 03_API.md, 06_OPERATIONS.md, 07_DEVELOPMENT.md, 09_INTEGRATIONS.md
