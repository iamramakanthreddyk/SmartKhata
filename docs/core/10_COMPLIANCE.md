# 10. SmartKhata - Compliance & Intellectual Property

**Purpose**: Legal compliance (GDPR, GST, IT Act), IP protection, contracts  
**Audience**: Founders, legal team, business team  
**Consolidates**: COMPLIANCE_FRAMEWORK.md, IP_PROTECTION.md

---

## 📋 GDPR Compliance (EU Users)

**Requirements** (for users in EU):

### Data Collection
- ✅ Explicit consent required before collecting personal data
- ✅ Clear privacy policy in simple language
- ✅ Legitimate basis documented (e.g., contract performance)

### User Rights
Users can request:
- **Access**: Copy of their data within 30 days
- **Rectification**: Correction of inaccurate data
- **Deletion**: "Right to be forgotten" (with exceptions)
- **Portability**: Export data in machine-readable format
- **Objection**: Stop processing for specific purposes

### Data Minimization
- Collect only data necessary for functionality
- Customer phone numbers only for invoices
- No tracking pixels or advertising cookies

### Data Protection
- HTTPS for all transmissions
- AES-256 encryption at rest
- Database encryption enabled
- Regular security audits (quarterly)

### Data Breach Notification
- Notify users within 72 hours of discovery
- Document breach in compliance log
- Report to relevant authority if >1000 affected

### Data Retention
- Active account data: Until business closes + 7 years (India law)
- Deleted account data: 30 days grace period, then permanent deletion
- Audit logs: 2 years
- Support tickets: 1 year

---

## 🇮🇳 GST Compliance (India)

**Registration Threshold**:
- Mandatory GST if turnover >₹40 lakhs/year
- Recommended for all businesses for compliance

### GSTR-1 Filing (Outward Supplies)
**Due**: 10th of next month

**Data Required**:
- Invoice number and date
- Party GSTIN or name
- Taxable value and tax amount
- Item descriptions

**Auto-Generation**:
- SmartKhata generates GSTR-1 automatically
- Pre-fills from created invoices
- User reviews and files via GST portal

### GSTR-2 Filing (Inward Supplies)
**Due**: 15th of next month

**Data Required**:
- Purchase invoices from suppliers
- Supplier GSTIN or name
- Taxable value and tax amount

**Process**:
- Users enter purchases in SmartKhata
- GSTR-2 auto-generated
- Pre-populated with e-invoiced purchases (ITC eligible)

### E-Invoicing
**Threshold**: From ₹5,00,000+ turnover (currently)

**Required**:
- All B2B invoices must be e-invoiced
- IRN (Invoice Registration Number) generated from GST portal API
- QR code on invoice
- Digital signature included

**Process**:
- SmartKhata generates invoice
- Sends to GST portal API
- Receives IRN in response
- Displays IRN on PDF

### E-Way Bills
**Threshold**: Consignment value >₹50,000

**Required** for:
- Stock transfers
- Goods delivery
- Interstate movements

**Process**:
- User enters shipment details
- SmartKhata generates e-way bill JSON
- Submits to e-way bill portal
- Returns e-way bill number

### GST Rate Structure (National, Not State-Based)

**Important**: GST rates are **NATIONAL and UNIFIED** across all states. Not state-dependent.

**Standard Rates**:
```
Rate    Category                    Examples
─────────────────────────────────────────────────────
0%      Essential goods             Food grains, medicines
5%      Basic goods                 Most retail items, staples
12%     Mid-tier goods              Electronics, textiles
18%     Common products             Cosmetics, services
28%     Luxury/Sin goods            Vehicles, alcohol, tobacco
```

**Intra-State vs Inter-State**:
- **Intra-state** (buyer & seller same state):
  - GST splits into CGST (Central) + SGST (State) equally
  - 18% → 9% CGST + 9% SGST
  - Filed separately in GSTR-1 (outward) vs GSTR-2 (inward)

- **Inter-state** (buyer & seller different states):
  - Single IGST (Integrated GST) rate
  - 18% → 18% IGST (no split)
  - Reverse charge may apply if unregistered buyer

### HSN/SAC Codes
**Requirement**:
- Suppliers with turnover >₹5Cr must mention HSN codes
- Traders must mention SAC codes
- SmartKhata provides HSN/SAC database

**Rate Determination**:
- Each HSN code has a **fixed official rate** (maintained by government)
- SmartKhata maintains this lookup table (updated annually)
- Rate cannot be arbitrary - only official rates allowed
- If HSN not in database: User must manually enter, but limited to 0/5/12/18/28%
- Rate affects GSTR filing and e-invoicing

### Late Filing Penalties
- GSTR-1 late filing: ₹100/day (max ₹5000)
- GSTR-2 late filing: ₹100/day (max ₹5000)
- No e-invoicing: Can reject input tax credit

---

## ⚖️ IT Act 2000 Compliance (India)

### Digital Signatures
**Requirement** (if annual turnover >₹10 crore):
- Invoices must be digitally signed
- Use Class 2 or Class 3 digital certificate

**SmartKhata Support**:
- Optional digital signature signing
- Integration with CA providers

### Invoice Standards
**Section 139A**:
- Invoice must contain:
  - Invoice number and date
  - Supplier details (name, address, GSTIN, PAN)
  - Customer details (name, address, GSTIN)
  - Description of goods/services
  - Quantity, rate, amount
  - Tax information

**SmartKhata Features**:
- ✅ All required fields included
- ✅ Auto-calculation of tax
- ✅ PDF generation with signatures
- ✅ Optional digital signature

### Records Retention
**Period**: 6 years

**Records Required**:
- All invoices (issued & received)
- Payment records
- GST returns filed
- Audit trails

**SmartKhata Support**:
- Backup of all invoices
- Export to PDF/CSV
- Audit trail logging

### Data Protection
**DPDP Act 2023**:
- Users are data principals
- Explicit consent for data processing
- Right to withdraw consent
- Data protection impact assessment

---

## 💼 IP Protection

### Trademarks
**SmartKhata® Registered Trademark**:
- Logo (wordmark + symbol)
- Registration: [To be filed with IP Registry]
- Coverage: Accounting software, business management

**Protection**:
- Use ® symbol on all materials
- Monitor for unauthorized use
- Send cease-and-desist for infringement

### Copyright
**Code & Documentation**:
- All source code: © SmartKhata LLC
- License: Proprietary (not open source)
- Duration: Life of author + 70 years

**User-Generated Content**:
- Invoices belong to business user
- SmartKhata retains license to store & process
- No commercial use by SmartKhata

### Patents
**Potential Patent Areas**:
- AI-based invoice reconciliation algorithm
- Offline-first synchronization method
- Smart expense categorization

**Strategy**:
- File provisional patents for core innovations
- Monitor competitor products
- Enforce if infringement detected

### Trade Secrets
**Protect**:
- Source code repository (private GitHub)
- Algorithm implementations (reconciliation, OCR)
- Customer data and business metrics
- Pricing models and financial data

**Protection Measures**:
- NDA required for all employees
- Code signing & integrity checks
- Encrypted backups
- Access control logs

---

## 📋 Legal Agreements

### Terms of Service
**Key Clauses**:
- Free tier conditions
- Premium subscription terms
- Data usage and retention
- Limitation of liability
- User responsibilities
- Dispute resolution

**Template**: [To be drafted by legal counsel]

### Privacy Policy
**Key Points**:
- What data we collect
- How we use it
- How long we keep it
- User rights (GDPR)
- Contact for privacy concerns

**Template**: [To be drafted by legal counsel]

### NDA (for employees & contractors)
**Covers**:
- Confidentiality of source code
- Protection of customer data
- Non-compete clause (12 months post-departure)
- IP ownership clause

**Template**: [Standard legal template]

### Data Processing Agreement (DPA)
**For GDPR Compliance**:
- Defines roles (controller vs processor)
- Sub-processor rules
- Data security standards
- Breach notification
- Data subject rights

---

## 🏢 Business Registration & Licensing

### Company Registration
- Type: Private Limited Company (LLC in US)
- Jurisdiction: India / Delaware
- GST Number: 06AABCT1234H1Z0 (sample)
- PAN: AAACT1234A (sample)

### Software Licensing
- No open source licenses used for proprietary code
- Third-party libraries: Comply with their licenses
- Attribution in code for open source dependencies

### Data Protection Registration
- Register with State Data Protection Officer
- Maintain breach register
- Conduct DPIAs annually

---

## 🚨 Compliance Checklist

### Legal Setup
- [ ] Company incorporated
- [ ] PAN & GSTIN registered
- [ ] Bank account opened
- [ ] Intellectual property registered (trademark)

### Data Protection
- [ ] Privacy policy published
- [ ] Terms of Service published
- [ ] Data Processing Agreement drafted
- [ ] GDPR compliance assessment done

### GST Compliance
- [ ] GSTR-1 filing system tested
- [ ] GSTR-2 filing system tested
- [ ] E-invoicing integration working
- [ ] E-way bill integration working

### Contracts
- [ ] Employee NDA prepared
- [ ] Contractor agreements ready
- [ ] Vendor agreements reviewed
- [ ] Customer T&Cs finalized

### Audits
- [ ] Code security review done
- [ ] Data access controls tested
- [ ] Encryption verified
- [ ] Backup procedures tested

### Monitoring
- [ ] Compliance calendar set (filing dates)
- [ ] Responsible person assigned
- [ ] Audit trail logging enabled
- [ ] Breach notification procedure ready

---

## 📅 Compliance Calendar

**Quarterly**:
- GST return filing (GSTR-1, GSTR-2, GSTR-3B)
- Security audit
- Data backup verification

**Semi-Annually**:
- DPIA review
- Vendor compliance audit
- Privacy policy review

**Annually**:
- Financial audit (India)
- Employee NDA renewal
- IP portfolio review
- Tax filing (ITR)

**As Needed**:
- Breach response
- Legal updates
- Competitor monitoring

---

## 🔗 Useful Links

**India**:
- GST Portal: https://www.gst.gov.in/
- E-Invoicing Portal: https://einvoice1.gst.gov.in/
- E-Way Bill Portal: https://ewaybillgst.gov.in/
- Ministry of Corporate Affairs: https://mca.gov.in/

**GDPR**:
- EU GDPR Guide: https://gdpr-info.eu/
- Data Protection Commission: https://edpb.europa.eu/

**IP**:
- IP Registry of India: https://ipindia.gov.in/
- USPTO: https://www.uspto.gov/

---

**Related Documents**: 04_SECURITY.md, 05_BUSINESS.md, 06_OPERATIONS.md
