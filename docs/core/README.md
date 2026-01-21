# SmartKhata Core Documentation

**Essential docs for building features correctly**

---

## ⚡ START HERE: DEVELOPER RULES

**→ [DEVELOPER_RULES.md](DEVELOPER_RULES.md)** ⭐ **READ THIS FIRST**
- The ONE Golden Rule (update, don't create)
- Enforcement mechanisms (git hook, PR checklist, Copilot)
- Red flags that get you blocked
- Examples of good vs. bad approaches

**Why?** This prevents 62 files from becoming 62 again.

---

## 📚 Which Doc Should I Read?

### **When starting a new feature:**
→ [HOW_TO_ADD_FEATURES.md](HOW_TO_ADD_FEATURES.md) 
- Exact workflow (3-phase discovery, then implement)
- Check what exists before coding
- Update only existing docs, don't create new ones
- Complete discount feature example

### **When designing database/API changes:**
→ [01_ARCHITECTURE.md](01_ARCHITECTURE.md)
- Database schema & all tables
- API layer design
- 6-layer architecture overview
- Naming conventions (snake_case DB, camelCase API)

### **When needing feature list:**
→ [02_FEATURES.md](02_FEATURES.md)
- All 38 features with descriptions
- What's implemented, what's planned
- Which layer owns each feature

### **When building REST endpoints:**
→ [03_API.md](03_API.md)
- All API endpoints (/api/v1, /api/v2)
- Request/response schemas
- Error codes & status
- Authentication requirements

### **When struggling with design decisions:**
→ [ARCHITECTURE_DECISION_MATRIX.md](ARCHITECTURE_DECISION_MATRIX.md)
- 10 key architecture decisions
- Pro/con for each choice
- When to use what pattern

### **When building for multiple platforms (iOS, Android, Web, Windows):**
→ [MULTI_FRONTEND_ARCHITECTURE.md](MULTI_FRONTEND_ARCHITECTURE.md)
- How shared types enable multi-platform
- Payment status feature example
- Monorepo structure

### **When debugging or stuck:**
→ [TROUBLESHOOTING.md](TROUBLESHOOTING.md)
- 8 common architecture problems
- Root cause for each
- How to fix

### **When adding custom prompts for Copilot:**
→ [COPILOT_PROMPTS.md](COPILOT_PROMPTS.md)
- Copy-paste prompts for features
- File map for discovery
- Exact commands to use

### **Reference - Security & Compliance:**
- [04_SECURITY.md](04_SECURITY.md) - Auth, JWT, RLS
- [10_COMPLIANCE.md](10_COMPLIANCE.md) - GST, legal
- [11_MAINTENANCE.md](11_MAINTENANCE.md) - Tech debt, upgrades

### **Reference - Operations & Business:**
- [05_BUSINESS.md](05_BUSINESS.md) - Pricing, freemium
- [06_OPERATIONS.md](06_OPERATIONS.md) - Deployment, monitoring
- [07_DEVELOPMENT.md](07_DEVELOPMENT.md) - Setup, tools

---

## 🎯 Quick Navigation

**I want to...**

| Goal | Read |
|------|------|
| Add a new feature | [HOW_TO_ADD_FEATURES.md](HOW_TO_ADD_FEATURES.md) |
| Understand the database | [01_ARCHITECTURE.md](01_ARCHITECTURE.md) |
| See all features | [02_FEATURES.md](02_FEATURES.md) |
| Build API endpoints | [03_API.md](03_API.md) |
| Use multiple frontends | [MULTI_FRONTEND_ARCHITECTURE.md](MULTI_FRONTEND_ARCHITECTURE.md) |
| Make a design decision | [ARCHITECTURE_DECISION_MATRIX.md](ARCHITECTURE_DECISION_MATRIX.md) |
| Fix an architecture problem | [TROUBLESHOOTING.md](TROUBLESHOOTING.md) |
| Use Copilot effectively | [COPILOT_PROMPTS.md](COPILOT_PROMPTS.md) |

---

## 🚀 Your First 30 Minutes

1. **Read** [HOW_TO_ADD_FEATURES.md](HOW_TO_ADD_FEATURES.md) (15 min)
   - Learn the workflow
   - See the discount feature example

2. **Skim** [01_ARCHITECTURE.md](01_ARCHITECTURE.md) (10 min)
   - Understand database schema
   - Know which layer does what

3. **Bookmark** [ARCHITECTURE_DECISION_MATRIX.md](ARCHITECTURE_DECISION_MATRIX.md) (5 min)
   - Reference when stuck

4. **Start coding** using HOW_TO_ADD_FEATURES.md workflow

---

## 📝 Core Files in This Folder

```
docs/core/
├── README.md ← You are here
├── 01_ARCHITECTURE.md (6-layer model, DB schema)
├── 02_FEATURES.md (all 38 features)
├── 03_API.md (REST endpoints)
├── 04_SECURITY.md (auth, RLS, compliance)
├── 05_BUSINESS.md (pricing, freemium model)
├── 06_OPERATIONS.md (deployment, monitoring)
├── 07_DEVELOPMENT.md (setup, tools, workflow)
├── 08_UX.md (mobile design principles)
├── 09_INTEGRATIONS.md (third-party APIs)
├── 10_COMPLIANCE.md (GST, legal)
├── 11_MAINTENANCE.md (tech debt, upgrades)
│
├── HOW_TO_ADD_FEATURES.md ⭐ (feature workflow - START HERE)
├── MULTI_FRONTEND_ARCHITECTURE.md (iOS/Android/Web/Windows)
├── ARCHITECTURE_DECISION_MATRIX.md (design decisions)
├── TROUBLESHOOTING.md (architecture problems & fixes)
├── COPILOT_PROMPTS.md (copy-paste prompts)
└── FILE_MAP.md (file structure reference)
```

---

## 🎓 Architecture Philosophy

**5 Core Principles** (Never break these):

1. **Mobile First** - Design for mobile, enhance for web
2. **No Duplication** - Single source of truth (check 3 docs before coding)
3. **Clear Separation** - Distinct layers (API, Services, Repos, DB)
4. **Easy Maintenance** - Simple, clear, documented
5. **Easy Expansion** - Add features without touching existing code

**LEGO Model**: Each module is self-contained. Removing one brick doesn't break others.

---

## ⚡ The One Rule

**Before you code, check 3 docs:**
1. [02_FEATURES.md](02_FEATURES.md) - Does it exist?
2. [01_ARCHITECTURE.md](01_ARCHITECTURE.md) - Where in database?
3. [ARCHITECTURE_DECISION_MATRIX.md](ARCHITECTURE_DECISION_MATRIX.md) - How to build it?

This prevents duplicate code and architectural mistakes.

---

## 📚 Documents

| Document | Purpose | Read Time | Audience |
|----------|---------|-----------|----------|
| [01_ARCHITECTURE.md](01_ARCHITECTURE.md) | System design, database schema, 6-layer architecture | 45 min | All technical team members |
| [02_FEATURES.md](02_FEATURES.md) | Complete feature specifications (38 features) | 60 min | Product, development, QA |
| [03_API.md](03_API.md) | REST API endpoints with examples (92 total) | 60 min | Backend & frontend developers |
| [04_SECURITY.md](04_SECURITY.md) | Authentication, RLS, compliance requirements | 45 min | Security, backend, DevOps |
| [05_BUSINESS.md](05_BUSINESS.md) | Pricing model, freemium strategy, financials | 45 min | Product, business, executives |
| [06_OPERATIONS.md](06_OPERATIONS.md) | Deployment, monitoring, scaling, incident response | 60 min | DevOps, SRE, infrastructure |
| [07_DEVELOPMENT.md](07_DEVELOPMENT.md) | Setup, testing, Git workflow, coding standards | 30 min | All developers |
| [08_UX.md](08_UX.md) | Mobile design, user flows, screen mockups | 60 min | Mobile developers, designers |
| [09_INTEGRATIONS.md](09_INTEGRATIONS.md) | Third-party APIs (GST, payments, messaging) | 45 min | Backend, integration teams |
| [10_COMPLIANCE.md](10_COMPLIANCE.md) | Legal, GDPR, GST, IP protection | 60 min | Legal, compliance, product |
| [11_MAINTENANCE.md](11_MAINTENANCE.md) | Technical debt, upgrades, performance tuning | 30 min | Tech leads, architects |
| [12_DOCUMENTATION_MAP.md](12_DOCUMENTATION_MAP.md) | Navigation guide and role-based reading paths | 10 min | Everyone |
| [PROJECT_STATUS.md](PROJECT_STATUS.md) | Current project status, next steps, development readiness | 15 min | All team members |

## 🔗 Cross-References

All core documents include "Related Documents" sections at the end linking to other relevant core documents.

## 📝 Maintenance

- **Single Source of Truth**: Each topic has one authoritative document
- **Version Control**: All documents include "Last Updated" dates
- **Consolidation**: No duplication between core documents
- **Updates**: Changes should be made to the appropriate core document

## 🎯 Reading Order

1. **Start**: [12_DOCUMENTATION_MAP.md](12_DOCUMENTATION_MAP.md) - Find your role-based path
2. **Foundation**: [01_ARCHITECTURE.md](01_ARCHITECTURE.md) - Understand the system
3. **Features**: [02_FEATURES.md](02_FEATURES.md) - What we're building
4. **Technical**: [03_API.md](03_API.md), [04_SECURITY.md](04_SECURITY.md), [07_DEVELOPMENT.md](07_DEVELOPMENT.md)
5. **Business**: [05_BUSINESS.md](05_BUSINESS.md) - Why we're building it

---

**Status**: ✅ Core documentation complete and organized  
**Last Updated**: January 20, 2026</content>
<parameter name="filePath">c:\Users\r.kowdampalli\Documents\MyProjects\SmartKhata\docs\core\README.md