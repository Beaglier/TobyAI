# Toby -- Project Master Context
**Version:** 1.0  
**Date:** April 2026  
**Owner:** Michael -- Senior Financial Controller  
**Status:** Pre-build. Schema complete. Awaiting entity incorporation and developer contract.

---

## 1. What Toby Is

Toby is an AI-native financial operating system built for Australian small and medium businesses. Toby replaces traditional accounting software by combining a deterministic financial engine -- which handles all accounting, compliance, payroll, job costing, and reporting with absolute accuracy -- with an AI interface that lets business owners and their teams interact with their finances in plain language. Rather than navigating menus and entering transactions manually, users can ask Toby questions, request reports, and action approvals through a conversational interface, while every number produced is calculated by rules-based logic with a complete audit trail behind it.

Toby manages the full financial lifecycle of a business: quoting, invoicing, accounts payable and receivable, payroll and superannuation, bank reconciliation, job and project costing, inventory, fixed assets, and compliance obligations including BAS, STP2, and TPAR -- all in one system, with no manual data re-entry between modules. For accountants and bookkeepers, Toby provides a dedicated partner portal giving multi-client oversight, BAS preparation tools, and management reporting. The system is built on a universal chart of accounts covering every industry from construction and trades to professional services, retail, healthcare, and agriculture.

---

## 2. Target Market

- **Primary:** Australian SMBs, $1-5M revenue, up to 20 staff
- **Geography:** Australia Phase 1. New Zealand Phase 2.
- **Channel:** Direct + accountant/bookkeeper reseller channel
- **Customer zero:** Any business -- not PCA-specific. Multi-tenant SaaS product.

---

## 3. Two-Layer Architecture

- **Layer 1 -- Deterministic engine:** All financial calculations, compliance rules, journal postings, report generation. No AI, no inference. Same input always produces same output.
- **Layer 2 -- AI interface:** Chat interface, Insights agent, FP&A agent, natural language queries. Reads from Layer 1 data. Never writes directly to the ledger.
- **Agents communicate via message queue** (AWS SQS / Azure Service Bus). No direct agent-to-agent calls. Every agent action produces a durable event log entry.

---

## 4. Key Decisions -- All Locked

### Product
- AU only Phase 1. NZ Phase 2. NZ fields in schema as nullable columns from day one.
- Universal COA (5-digit codes, ~391 accounts). No tenant ever needs to add an account.
- Tenants can rename accounts and activate/deactivate via Admin settings. Cannot change codes, types, or FS line mappings.
- Four active dimensions (Job, Region, Department, Cost Type) + four additional active (Asset Class, Cost Phase, Payroll Category, Customer Segment) + four dormant (Award, Cost Code, Contract Type) + Funding Source (activated Phase 1 for financial accounts).
- Option B auto-tagging: rules-based pre-fill + learn mode. All four dimensions can tag simultaneously.
- Eight user types: Owner/Admin, Supervisor (FC/FP&A), Finance Officer, Payroll Officer, Site Supervisor, Read Only, Employee Self-Service (free), Accountant Partner.
- Job type configuration: eight industry skins (Construction, Professional Services, Creative/Agency, IT/Consulting, Trades, Manufacturing, Healthcare, Other). Same schema, different field labels.

### Payroll
- **Phase 1: integrate with Employment Hero / KeyPay API.** Payroll calculation, STP2, SuperStream, leave engine handled by partner. Toby receives payroll journal events and auto-posts to ledger.
- Native payroll (Phase 2) once STP2 DSP registration is approved and business is generating revenue.
- STP2 DSP registration to be initiated immediately despite Phase 1 integration approach -- 3-6 month lead time.

### Build
- Hybrid onshore/offshore model. Senior developer onshore owns compliance, architecture, security. Offshore for UI/workflow in later phases.
- AI-assisted development throughout. Realistic productivity multiplier: 1.4-1.6x. Not 3-5x.
- Target: beta at month 5-6. Early adopter launch (900 seats) at month 9-10. GA at month 12-14.

### Pricing
- Modular per-user per-month SaaS. Anchored below Xero + bookkeeper cost.
- Three cohorts: 100 beta (3-month free trial, 50% retail for life), 900 early access (3-month free trial, full retail), GA open signup.
- Accountant wholesale: 35% below retail.
- Employee self-service: free per seat.

### Commercial
- Build cost estimate: ~$155-165K at optimised hybrid rates (onshore $135/hr, offshore $55/hr) with AI assistance potentially compressing to lower end.
- Toby entity to be incorporated separately from PCA Ground Engineering.
- IP ownership in Toby entity. Developer contract must include IP assignment, no reuse rights, repository access revocation on termination, milestone payments.

### Data and Compliance
- Michael owns all AU compliance rule inputs. Developer never specifies compliance rules independently.
- All rates and rules in versioned configuration tables, never hardcoded.
- Soft delete everywhere. Effective dating on all rate tables. Audit trail on every table.
- Seven-year data retention. Tenant isolation at row level with RLS at database layer.
- Currency field on every monetary amount from day one (Phase 1 = AUD only).
- Timezone: UTC storage, tenant local display, compliance deadlines in AEST.

### Sub-ledgers
- Option B: control accounts in COA, detail in sub-ledger records.
- Financial Account Card is the sub-ledger for bank accounts, loans, credit cards, director loans.
- Every journal line to a financial control account requires financial_account_id FK. Enforced at posting engine.
- AR sub-ledger = customer card balances. AP sub-ledger = vendor card balances.
- All sub-ledgers reconcile to control accounts as a blocking step in month-end close.

### Journals
- Seven types: Manual, Recurring, Reversing, System, Opening, Adjustment, Intercompany (dormant).
- All manual journals require Supervisor approval. Cannot self-approve. Four-eyes enforced.
- System journals auto-post but reviewed by Audit agent.
- Recurring journals queue for approval -- never auto-post.
- Period locking: Open, Soft Closed, Hard Locked, Future.
- 25 standard journal templates provisioned at tenant onboarding.

### AASB / IFRS
- Dual depreciation tracking: tax schedule (ATO effective life) and accounting schedule (entity-assessed useful life).
- AASB 16 leases: ROU asset and lease liability capitalised. Auto-journalled by Fixed Asset agent.
- AASB 15 revenue recognition: percentage of completion for project revenue. Contract asset/liability tracked at project level.
- AASB 9 ECL: simplified approach for trade receivables. Quarterly provision by aging bucket.
- AASB 112 deferred tax: FP&A agent generates schedule. Manual posting by accountant.
- Deferred tax accounts: 15021 DTA current, 18090 DTA non-current, 22041 DTL current, 26040 DTL non-current.

### WIP
- WIP auto-posting at month-end. FP&A agent calculates and posts to 14010 (billable time) and 14011 (materials).
- WIP clears automatically when invoice raised against time entries.
- WIP write-off: manual, Supervisor approved, posts to 76090.

### Quotes
- Internal Supervisor approval required before quote sent to client. Cannot self-approve.
- Eight line types: Labour-Employee, Labour-Type, Plant/Equipment (Resource card), Materials (Inventory card), Subcontractor (Vendor card), Expense, Overhead, Discount.
- Accepted quote auto-creates Project card and populates budget from quote cost values.
- Resource availability checking: Phase 2.

### Reporting
- All standard reports pre-aggregated. Target: under 3 seconds.
- Every figure drills down to source transactions.
- Output formats: on-screen, Excel, PDF.
- 28 standard reports defined in Report Engine sheet.

---

## 5. Document Inventory

All documents produced during this project:

| Document | Version | Description |
|---|---|---|
| Toby_Business_Case_v3.0.xlsx | v3.0 | P&L, unit economics, scenario modelling, build cost |
| Toby_Default_COA_v3.0.xlsx | v3.0 (master schema) | 42-sheet complete pre-build schema package |
| Toby_Universal_COA_v1.0.xlsx | v1.0 | Universal 5-digit COA, 391 accounts, all industries |

### Toby_Default_COA_v3.0.xlsx -- Sheet Inventory (42 sheets)

1. **Chart of Accounts** -- 171 accounts (legacy 4-digit, superseded by Universal COA)
2. **Mandatory Field Policy** -- Three-tier enforcement (Creation / Transaction / Payment)
3. **Dimensions v2** -- 12 dimensions (8 active, 4 dormant), asset class table, payroll category table, cost phase table
4. **Config Tables** -- Super rates FY21-26, state payroll tax, ATO effective lives, NMW, Fair Work award structure, corporate tax rate, ECL rates, accounting useful lives
5. **Structural Architecture** -- 12 non-negotiable build requirements (soft delete, effective dating, audit trail, tenant isolation, currency awareness, timezone, dimension extensibility, event-driven agents, report caching, configuration versioning, four-eyes payment, data retention)
6. **AASB IFRS Reference** -- 8 standards (AASB 116, 16, 15, 102, 137, 136, 112, 9), dual depreciation examples, AASB 16 journal structure, ECL provision matrix
7. **Dimensions Schema** -- Original dimension schema (superseded by Dimensions v2)
8. **Reporting Matrix** -- 15 reports mapped against dimensions
9. **Auto-Tag Rules** -- 7 trigger types in priority order, account-code rules, supplier type defaults, UI behaviour spec, learn mode
10. **Dimension Dev Notes** -- 11 implementation notes + jurisdiction expansion architecture + auto-tagging engine notes
11. **Account Ranges** -- 4-digit range structure (superseded by Universal COA)
12. **GST Treatment Guide** -- 8 GST codes AU/NZ
13. **Developer Notes (COA)** -- 15 implementation notes
14. **Vendor Card** -- 70+ fields: identification, contact, tax/compliance, TPAR (AU), schedular payments (NZ Phase 2), financial defaults, banking (encrypted), dimensions, document preferences
15. **Customer Card** -- 65+ fields: identification, contact, tax/compliance, credit/payment terms, dunning, invoicing preferences, dimensions
16. **Card Dev Notes** -- 10 notes: TPAR accumulation, credit hold, dunning workflow, banking security, duplicate detection, card merge
17. **Employee Card** -- 120+ fields: identification, contact, TFN (AU encrypted), employment, remuneration (restricted/historical), allowances/deductions, leave, super (AU), bank (encrypted), STP2 (AU), Payday Filing (NZ Phase 2), project costing/timesheet
18. **Employee Dev Notes** -- 10 notes: TFN encryption, remuneration history, leave accrual engine, STP2, Payday Filing, SuperStream, termination pay, self-service portal, award interpretation, privacy/retention
19. **Project Card** -- 60+ fields: identification, client/contract, dates, financial budget by cost category, revenue/billing (AASB 15), team/resources, site location, compliance, history
20. **Inventory Card** -- 50+ fields: identification, UOM, financial accounts, FIFO valuation, purchasing, stock control, project costing integration
21. **Resource Card** -- 55+ fields, 4 types (Plant, Equipment, Tool, Labour Type): identification, plant/equipment details, compliance/certification, labour type, costing/scheduling, availability
22. **Master Data Dev Notes** -- 10 notes: JOB dimension sync, budget alerts, revenue recognition, retention, FIFO engine, three-way match, stocktake, plant on jobs, labour type planning, card relationships FK map
23. **Journal Engine** -- Header schema (27 fields), line schema (23 fields, all 8 dimension FKs), 7 journal types, period management, recurring schedule schema, approval workflow, 25 standard templates, 15-step month-end close sequence
24. **Journal Dev Notes** -- 10 notes: double-entry enforcement, state machine, posting engine, recurring schedule engine, reversing automation, period locking, search/FTS, template versioning, voiding, system journal anomaly detection
25. **User Types and Permissions** -- 8 user types, 100+ permissions across 11 modules, user card schema, delegation of authority (12 transaction types), accountant partner special rules
26. **User Security Dev Notes** -- 10 notes: RBAC 3 layers, MFA enforcement, session management, password policy, account lockout, delegation limit enforcement, accountant portal isolation, employee self-service isolation, invitation flow, sensitive data access logging
27. **Job Type Config** -- 8 industry skins, field label mapping table (12 fields relabelled by job type)
28. **Quote Card** -- Header (identification, client, dates, internal approval, financial summary incl internal margin, terms, outcome), line schema (25 fields, 8 line types), status workflow (9 statuses), quote-to-project conversion (18 auto-populated fields)
29. **Timesheet Module** -- Header + 24-field entry lines, configurable approval chains by job type, WIP auto-posting (4-step), 8 utilisation metrics, activity codes by job type (15 codes)
30. **Sub-Ledger Architecture** -- 16 control account mappings, 8 reconciliation rules (blocking vs informational)
31. **Financial Account Card** -- identification (11 account types), institution details, bank feed/rec, loan facility (amortisation fields), director/related party loans, dimension defaults, 8 auto-generated journals
32. **Financial Account Dev Notes** -- 10 notes: control account integrity, mandatory FK, loan amortisation formula, credit card sign convention, director loan direction/FBT, related party AASB 124, current/non-current reclassification, multi-currency Phase 2, Funding dimension activation, sub-ledger reconciliation workflow
33. **BAS GST Engine** -- G-label mapping (G1-G19, 1A, 1B), W-label mapping (W1-W5), 9-step BAS workflow, BAS record schema
34. **Budget Forecast Engine** -- Budget header and line schemas, 6 entry methods, 4 forecast methods, 5 variance report types
35. **Audit Trail Schema** -- 20-field event schema, 25-event taxonomy (Financial/Security/Config/Compliance), retention and export rules
36. **Compliance Calendar** -- 19 obligations with deadlines, alert lead times, consequences, tracking schema
37. **Tenant Settings** -- 60+ configurable fields across 6 sections (identity, financial year, payroll, approvals, alerts, documents)
38. **Data Migration** -- 16-step onboarding sequence, 11 data types with source/format/validation, 11 validation rules
39. **Notification Engine** -- 17-field schema, 24 notification types with severity/channel/escalation/action
40. **Report Engine** -- 28 standard reports, generation architecture (pre-aggregation, caching, drill-down, Excel/PDF)
41. **Error Handling** -- 4 error classes (Validation/Business Rule/Integration/System), 10 message examples (bad vs good), 8 resilience patterns
42. **Internal API Spec** -- 12 API conventions, 35 core endpoints across all modules

### Toby_Universal_COA_v1.0.xlsx -- Sheet Inventory (4 sheets)

1. **Universal COA** -- 391 accounts, 5-digit codes (10000-89999), 15 columns per account
2. **COA Range Guide** -- Complete range structure with activation type and purpose
3. **Activation Guide** -- Module activation table (11 modules), Industry tag activation (17 industry tags)
4. **Admin Config Guide** -- What tenants can/cannot change, naming conventions, COA versioning

---

## 6. COA Account Code Structure (5-digit)

| Range | Section |
|---|---|
| 10000-19999 | Assets |
| 11000-11999 | Cash and cash equivalents |
| 12000-12999 | Receivables |
| 13000-13999 | Inventories |
| 14000-14999 | WIP -- projects |
| 15000-15999 | Prepayments and other current assets |
| 16000-16999 | PPE and ROU assets |
| 17000-17999 | Intangible assets |
| 18000-18999 | Investments and non-current financial assets |
| 20000-29999 | Liabilities |
| 21000-21999 | Current payables |
| 22000-22999 | Tax liabilities |
| 23000-23999 | Payroll liabilities |
| 24000-24999 | Short-term borrowings |
| 25000-25999 | Provisions -- current |
| 26000-26999 | Non-current liabilities |
| 27000-27999 | Provisions -- non-current |
| 30000-39999 | Equity |
| 40000-49999 | Revenue |
| 50000-59999 | Cost of sales |
| 60000-69999 | Operating expenses |
| 70000-74999 | Other income |
| 75000-79999 | Other expenses |
| 80000-89999 | Clearing and statistical |

---

## 7. Technology Stack (Confirmed Decisions)

- **Backend:** Python
- **Database:** Managed cloud database, Azure Australia East
- **Bank feeds:** Basiq
- **Payroll Phase 1:** Employment Hero / KeyPay API integration
- **FX rates:** RBA daily exchange rates, transaction-date locking
- **Message queue:** AWS SQS or Azure Service Bus (event-driven agent architecture)
- **Development model:** Hybrid onshore/offshore, known contractor, hourly rate

---

## 8. Compliance Ownership Rules

Michael (Senior Financial Controller, CA) is the authoritative source on all AU compliance rules. The developer never specifies compliance logic independently. Specific ownership:

- PAYG withholding tables and rates
- Award interpretation and minimum rates
- Superannuation guarantee rules and rates
- STP2 schema and lodgement requirements
- BAS label calculations and GST treatment rules
- TPAR reportable industries and lodgement format
- Payroll tax thresholds by state
- ATO effective life determinations
- AASB standard interpretations for SMB context
- Leave entitlement rules by state (AU) and employment type

---

## 9. Architecture Principles (Non-Negotiable)

1. **Deterministic outputs.** Every calculation produces the same result given the same input. No probabilistic or AI-inferred numbers in financial outputs.
2. **Fail loudly.** Missing data, malformed inputs, out-of-tolerance values -- stop and alert. Never silently produce a wrong number.
3. **Configuration over code.** All business rules (tax tables, award rates, thresholds, mappings) in configuration records. No code deployment to update a rate.
4. **Separation of concerns.** Data ingestion, transformation, business logic, and presentation are distinct layers. Never mixed.
5. **Immutable audit trail.** Every record change logged with before/after state. Append-only. Seven-year retention.
6. **Soft delete everywhere.** No hard deletes. ever.
7. **Effective dating on all rates.** Every rate table has valid_from and valid_to. Historical calculations always reproducible.
8. **Tenant isolation.** Row-level tenant_id on every table. RLS at database layer. No cross-tenant data leakage.
9. **Four-eyes on money.** Payment runs, payroll, and journal voids require two distinct users.
10. **Event-driven agents.** All agents communicate via message queue. No direct agent-to-agent calls. No event silently dropped.

---

## 10. Pre-Build Blockers (Things That Must Happen Before Build Starts)

| Item | Owner | Urgency | Status |
|---|---|---|---|
| Incorporate Toby entity | Michael + lawyer | Critical | Not done |
| Draft and sign developer contract | Michael + lawyer | Critical | Not done |
| Equity conversation with developer | Michael | Critical | Not done |
| Confirm Employment Hero / KeyPay partnership terms | Michael | High | Not done |
| Initiate STP2 DSP registration | Michael | High -- long lead time | Not done |
| COA sign-off (Universal COA v1.0) | Michael | High | Ready for review |
| Delegation authority matrix | Michael | High | Not done |
| User roles matrix (named individuals) | Michael | High | Not done |
| Basiq account and API credentials | Michael | Medium | Not done |
| Founding accountant partner conversations (3-5 firms) | Michael | Medium | Not done |

---

## 11. Recommended 30-Day Action Sequence

**Week 1**
- Engage lawyer: incorporate Toby entity, draft developer contract, prepare IP assignment
- Have equity conversation with developer before contract is drafted
- Initiate STP2 DSP registration
- Contact Employment Hero / KeyPay partnership team

**Week 2**
- Sign developer contract
- Review and sign off Universal COA v1.0
- Produce delegation authority matrix
- Produce user roles matrix

**Week 3**
- Developer produces architecture document (first contractual deliverable)
- Define pay run groups and payroll integration configuration
- Begin founding accountant partner conversations (target 3-5 firms committed before beta)

**Week 4**
- Independent technical review of architecture document
- Employment Hero / KeyPay API access confirmed
- Phase 1 build commences

---

## 12. Build Timeline (with AI-Assisted Development)

| Milestone | Month |
|---|---|
| Phase 1 core (COA, bank rec, AP, AR, BAS, journals) | Month 3-4 |
| Payroll integration (Employment Hero / KeyPay) live | Month 4-5 |
| Closed beta -- 50 tenants | **Month 5-6** |
| Fixed assets, inventory, project costing added | Month 7-8 |
| Early adopter launch -- 900 seats | **Month 9-10** |
| Full system with AI interface and FP&A | Month 11-12 |
| GA | **Month 12-14** |

---

## 13. What a New Claude Instance Needs to Know

When continuing this project in a new Claude instance:

1. Paste this file at the start of the conversation
2. Reference specific schema sheets by name when discussing technical decisions
3. Michael owns all compliance rule inputs -- Claude should never specify AU tax/payroll rules without Michael confirming them
4. All financial outputs must be deterministic. Never suggest probabilistic or AI-inferred approaches for financial calculations.
5. The Universal COA (5-digit) is the authoritative account code reference. The 4-digit COA in the schema package is legacy.
6. Phase 1 = AU only. NZ fields exist in schema as nullable columns. Never remove them.
7. Payroll Phase 1 = Employment Hero / KeyPay integration. Native payroll is Phase 2.
8. This is a commercial SaaS product, not an internal tool for PCA Ground Engineering.

---

## 14. Glossary

| Term | Meaning |
|---|---|
| Toby | The product -- AI-native financial OS |
| Layer 1 | Deterministic rules engine -- owns all financial calculations |
| Layer 2 | AI interface -- presents Layer 1 data, never writes to ledger directly |
| DSP | Digital Service Provider -- ATO registration required for STP2 |
| STP2 | Single Touch Payroll Phase 2 -- ATO payroll reporting |
| SG | Superannuation Guarantee -- employer super contribution |
| TPAR | Taxable Payments Annual Report -- ATO contractor payment reporting |
| BAS | Business Activity Statement -- quarterly GST/PAYG report to ATO |
| ROU | Right-of-Use asset -- AASB 16 leased asset |
| WIP | Work in Progress -- unbilled billable time and costs |
| ECL | Expected Credit Loss -- AASB 9 doubtful debts provision |
| DTA / DTL | Deferred Tax Asset / Deferred Tax Liability -- AASB 112 |
| ITC | Input Tax Credit -- GST claimable on purchases |
| FIFO | First In First Out -- inventory valuation method |
| RLS | Row Level Security -- database-level tenant isolation |
| JWT | JSON Web Token -- authentication token |
| COA | Chart of Accounts |
| FP&A | Financial Planning and Analysis |

---

*This document was generated from the Toby project development session. Keep it updated as decisions are made and milestones are reached. Version-control it alongside the schema package.*

---

## 15. Universal COA -- Key Accounts Reference (5-Digit)

### Assets (10000-19999)
| Code | Account Name | Type | Core/Opt | Module |
|---|---|---|---|---|
| 11010 | Cash at Bank -- Operating | Asset | Core | BANK |
| 11011 | Cash at Bank -- Savings / Reserve | Asset | Core | BANK |
| 11013 | Cash at Bank -- Trust | Asset | Optional | BANK |
| 12010 | Accounts Receivable -- Trade | Asset | Core | AR |
| 12011 | Allowance for Doubtful Debts | Asset | Core | AR |
| 12012 | Retention Receivable | Asset | Optional | AR/PROJECTCOSTING |
| 12013 | Unbilled Revenue (Contract Asset) | Asset | Optional | AR/PROJECTCOSTING |
| 12014 | GST Receivable (Input Tax Credits) | Asset | Core | CORE |
| 13013 | Merchandise / Stock on Hand | Asset | Optional | INVENTORY |
| 14010 | WIP -- Billable Time (uninvoiced) | Asset | Optional | TIMESHEETS |
| 14011 | WIP -- Direct Costs (uninvoiced) | Asset | Optional | PROJECTCOSTING |
| 15010 | Prepaid Expenses | Asset | Core | CORE |
| 15021 | Deferred Tax Asset -- Current | Asset | Optional | FPA |
| 16040 | Plant and Equipment (at cost) | Asset | Optional | FIXEDASSETS |
| 16041 | Accum Depreciation -- Plant and Equipment | Asset | Optional | FIXEDASSETS |
| 16510 | ROU Asset -- Property Leases | Asset | Optional | FIXEDASSETS |
| 16511 | Accum Depreciation -- ROU Property | Asset | Optional | FIXEDASSETS |
| 17010 | Goodwill | Asset | Optional | FIXEDASSETS |
| 17050 | Software (at cost) | Asset | Optional | FIXEDASSETS |
| 18010 | Investments -- Listed Equities | Asset | Optional | FPA |
| 18040 | Investment Property | Asset | Optional | FIXEDASSETS |
| 18090 | Deferred Tax Asset -- Non-Current | Asset | Optional | FPA |
| 18100 | Security Bonds and Deposits | Asset | Optional | CORE |

### Liabilities (20000-29999)
| Code | Account Name | Type | Core/Opt | Module |
|---|---|---|---|---|
| 21010 | Accounts Payable -- Trade | Liability | Core | AP |
| 21011 | Accrued Expenses | Liability | Core | CORE |
| 21012 | Deferred Revenue -- Current | Liability | Optional | AR |
| 21013 | Contract Liabilities -- Current | Liability | Optional | PROJECTCOSTING |
| 22010 | GST Payable (Output Tax) | Liability | Core | CORE |
| 22011 | GST Clearing | Liability | Core | CORE |
| 22020 | PAYG Withholding Payable | Liability | Optional | PAYROLL |
| 22030 | Payroll Tax Payable | Liability | Optional | PAYROLL |
| 22040 | Income Tax Payable -- Current | Liability | Core | CORE |
| 22041 | Deferred Tax Liability -- Current | Liability | Optional | FPA |
| 23010 | Wages and Salaries Payable | Liability | Optional | PAYROLL |
| 23020 | Superannuation Payable | Liability | Optional | PAYROLL |
| 23030 | Annual Leave Provision | Liability | Optional | PAYROLL |
| 23031 | Personal / Sick Leave Provision | Liability | Optional | PAYROLL |
| 23032 | Long Service Leave Provision -- Current | Liability | Optional | PAYROLL |
| 24010 | Bank Overdraft | Liability | Optional | BANK |
| 24020 | Credit Cards Payable | Liability | Optional | BANK |
| 24040 | Current Portion -- Long-Term Loans | Liability | Optional | BANK |
| 24041 | Current Portion -- Finance Lease Liabilities | Liability | Optional | FIXEDASSETS |
| 24042 | Current Portion -- Director Loans | Liability | Optional | BANK |
| 25010 | Warranty Provision | Liability | Optional | CORE |
| 25020 | Onerous Contract Provision | Liability | Optional | CORE |
| 25040 | Legal Claims Provision | Liability | Optional | CORE |
| 26010 | Long-Term Loans -- Bank | Liability | Optional | BANK |
| 26020 | Director Loans -- Non-Current | Liability | Optional | BANK |
| 26030 | Finance Lease Liabilities -- Non-Current | Liability | Optional | FIXEDASSETS |
| 26040 | Deferred Tax Liability -- Non-Current | Liability | Optional | FPA |
| 27010 | Long Service Leave Provision -- Non-Current | Liability | Optional | PAYROLL |
| 27020 | Make-Good Provision | Liability | Optional | CORE |

### Equity (30000-39999)
| Code | Account Name | Type | Core/Opt |
|---|---|---|---|
| 31010 | Paid-Up Capital / Share Capital | Equity | Core |
| 31040 | Owner's Equity / Capital Account | Equity | Optional |
| 32010 | Retained Earnings | Equity | Core |
| 32020 | Current Year Earnings | Equity | Core |
| 32030 | Owner's Drawings | Equity | Optional |
| 32040 | Dividends Declared | Equity | Optional |
| 33010 | Foreign Currency Translation Reserve | Equity | Optional |
| 33020 | Asset Revaluation Reserve | Equity | Optional |

### Revenue (40000-49999)
| Code | Account Name | GST | Core/Opt | Industry |
|---|---|---|---|---|
| 41010 | Sales -- Products (taxable) | STD | Core | RETAIL/MANUFACTURING |
| 41020 | Sales -- Services (taxable) | STD | Core | ALL |
| 41021 | Sales -- Services (GST-free) | FREE | Optional | HEALTHCARE/EDUCATION |
| 41030 | Project Revenue | STD | Optional | ALL |
| 41031 | Progress Claim Revenue | STD | Optional | CONSTRUCTION |
| 41040 | Subscription and Recurring Revenue | STD | Optional | ALL |
| 41050 | Professional Fees | STD | Optional | PROFSERVICES |
| 41060 | Labour Hire and Contracting Revenue | STD | Optional | LABOURHIRE |
| 41070 | Rental and Hire Income | STD | Optional | REALESTATE/EQUIPMENT |
| 41071 | Residential Rental Income | ITAX | Optional | REALESTATE |
| 41120 | Clinical and Patient Fees | FREE | Optional | HEALTHCARE |
| 41122 | Medicare Benefits | FREE | Optional | HEALTHCARE |
| 41160 | Agricultural Sales -- Livestock | FREE | Optional | AGRICULTURE |
| 41171 | Government Grants -- Recurrent | FREE | Optional | ALL |
| 42010 | Sales Returns and Allowances | STD | Core | ALL |
| 42020 | Discounts Given | STD | Core | ALL |

### Cost of Sales (50000-59999)
| Code | Account Name | Core/Opt | Industry |
|---|---|---|---|
| 51010 | Cost of Goods Sold | Core | RETAIL/MANUFACTURING |
| 51011 | Purchases -- Stock | Optional | RETAIL/WHOLESALE |
| 51020 | Direct Labour -- Wages | Optional | ALL |
| 51030 | Direct Labour -- Projects | Optional | ALL |
| 51040 | Direct Materials -- Projects | Optional | ALL |
| 51050 | Subcontractor Costs | Optional | ALL |
| 51060 | Equipment Hire -- Projects | Optional | CONSTRUCTION/TRADES |
| 51070 | Clinical and Medical Supplies | Optional | HEALTHCARE |
| 51100 | Event and Production Costs | Optional | EVENTS/CREATIVE |
| 51110 | Disbursements -- Rechargeable | Optional | PROFSERVICES |

### Operating Expenses (60000-69999)
| Code | Account Name | Deductible | Core/Opt |
|---|---|---|---|
| 61010 | Wages and Salaries | Yes | Core |
| 61020 | Superannuation -- Employer | Yes | Optional |
| 61030 | Payroll Tax | Yes | Optional |
| 61040 | Workers Compensation Insurance | Yes | Optional |
| 61060 | Staff Recruitment and Onboarding | Yes | Optional |
| 61070 | Training and Development | Yes | Optional |
| 61082 | Christmas and Staff Events | No | Optional |
| 61100 | Directors Fees | Yes | Optional |
| 62010 | Rent and Lease Payments | Yes | Core |
| 62011 | Lease Interest Expense (AASB 16) | Yes | Optional |
| 63010 | Motor Vehicle Fuel | Yes | Optional |
| 64010 | Advertising and Marketing | Yes | Optional |
| 64030 | Entertainment and Client Gifts (50%) | Yes 50% | Optional |
| 64050 | Sponsorship | Yes | Optional |
| 65010 | Accounting and Tax Fees | Yes | Core |
| 65011 | Audit Fees | Yes | Optional |
| 65012 | Legal Fees | Yes | Optional |
| 65020 | Bank Charges and Merchant Fees | Yes | Core |
| 65030 | Software Subscriptions (SaaS) | Yes | Core |
| 65033 | Cybersecurity | Yes | Optional |
| 65040 | Telephone and Internet | Yes | Core |
| 65090 | General Insurance | Yes | Optional |
| 65100 | Licences and Registrations | Yes | Optional |
| 65120 | Donations and Community Investment | No | Optional |
| 65121 | Donations to DGR | Yes | Optional |
| 65140 | Research and Development Expense | Yes | Optional |
| 66010 | Depreciation -- Buildings | Yes | Optional |
| 66020 | Depreciation -- Plant and Equipment | Yes | Optional |
| 66060 | Depreciation -- ROU Assets | N/A | Optional |
| 66070 | Amortisation -- Intangible Assets | Yes | Optional |
| 67010 | Interest Expense -- Bank Loans | Yes | Optional |
| 67011 | Interest Expense -- Finance Leases | Yes | Optional |

### Other Income (70000-74999)
| Code | Account Name | GST | Core/Opt |
|---|---|---|---|
| 71010 | Interest Income | ITAX | Core |
| 71020 | Dividend Income | ITAX | Optional |
| 71050 | Profit on Disposal of Assets | STD | Optional |
| 71060 | Foreign Exchange Gain | NA | Optional |
| 71070 | Government Subsidies and Incentives | FREE | Optional |
| 71080 | Insurance Recoveries | STD | Optional |

### Other Expenses (75000-79999)
| Code | Account Name | Deductible | Core/Opt |
|---|---|---|---|
| 76010 | Loss on Disposal of Assets | N/A | Optional |
| 76030 | Bad Debts Written Off | Yes | Optional |
| 76031 | Bad Debt Expense (ECL Provision) | Yes | Optional |
| 76040 | Impairment Loss -- Goodwill | No | Optional |
| 76050 | Penalties and Fines | No | Optional |
| 76090 | WIP Write-off | Yes | Optional |
| 77010 | Income Tax Expense -- Current | N/A | Core |
| 77011 | Income Tax Expense -- Deferred | N/A | Optional |
| 77020 | FBT Expense | No | Optional |

### Clearing (80000-89999)
| Code | Account Name | Core/Opt |
|---|---|---|
| 81010 | Suspense Account | Core |
| 81020 | Rounding Account | Core |
| 81030 | Payroll Clearing | Optional |
| 81050 | Opening Balance Adjustment | Core |
| 81060 | GST Clearing | Core |

---

## 16. Dimensions -- Full Reference

### Active Dimensions (Phase 1)

| Code | Name | Prefix | Mandatory | Purpose |
|---|---|---|---|---|
| JOB | Job / Project | JOB- | When Project Costing active | Links transactions to jobs. JOB-OVERHEAD always available. |
| REGION | Region / Location | REG- | Never | Geographic split. States, sites, territories. |
| DEPT | Department / Cost Centre | DEPT- | Never | Organisational split. Ops, Admin, Sales, Finance. |
| COST | Cost Type | COST- | Never | Economic classification. Labour, Materials, Overhead etc. |
| ASSET | Asset Class | ASSET- | When Fixed Assets active | Groups assets for depreciation engine. |
| PHASE | Cost Phase | PHASE- | Never | Project lifecycle. Tender, Mobilise, Construct, Commission, Defects. |
| PAYCAT | Payroll Category | PAYCAT- | System-assigned on payroll | STP2 disaggregation. OTE, Overtime, Leave, Allowance, Super. |
| CUSTSEG | Customer Segment | SEG- | Never | Revenue analysis by customer type. |

### Dormant Dimensions (in schema, not in Phase 1 UI)

| Code | Name | When to Activate |
|---|---|---|
| AWARD | Award / Agreement | Phase 2 -- when native payroll built |
| COSTCODE | Cost Code | When industry cost code benchmarking needed |
| CONTRACT | Contract Type | Head contract / subcontract / variation analysis |

### Funding Source Dimension (Active Phase 1 -- financial accounts)

| Code | Name | Use |
|---|---|---|
| FUND-EQUITY | Equity Funded | Owner equity / retained earnings |
| FUND-DEBT | Debt / External Borrowing | Bank loans, finance company |
| FUND-DIRECTOR | Director / Shareholder Funded | Director and shareholder loans |
| FUND-GRANT | Grant Funded | Government and private grants |
| FUND-OPERATING | Self-Funded | Day-to-day operating cash flow |

### Default Cost Type Values

| Code | Name | Used For |
|---|---|---|
| COST-LABOUR | Labour | All employee and contractor time |
| COST-MATERIALS | Materials | Stock, consumables, raw materials |
| COST-SUBCON | Subcontractors | Third-party contractors |
| COST-OVERHEAD | Overhead | Indirect costs, administration |
| COST-TRAVEL | Travel and Accommodation | Travel, accommodation, transport |
| COST-EQUIP | Equipment Hire | Plant and equipment hire |
| COST-FREIGHT | Freight and Logistics | Inbound and outbound freight |
| COST-MARKETING | Marketing | Marketing and sales costs |
| COST-PROF | Professional Services | Accounting, legal, consulting |

---

## 17. User Types -- Permission Summary

| Role | Billing | Can Approve | Can Post | Key Restrictions |
|---|---|---|---|---|
| Owner / Administrator | Yes | All | All + adjustments + voids | None |
| Supervisor (FC/FP&A) | Yes | Journals, payments, payroll, invoices | All except adjustments/voids | Cannot void journals |
| Finance Officer | Yes | None -- submits only | Cannot post independently | Cannot view salary data |
| Payroll Officer | Yes | None -- submits pay runs | Cannot post independently | Payroll and employee data only |
| Site Supervisor | Yes | Own crew timesheets only | None | Assigned project costs read only |
| Read Only | Yes | None | None | Reports and dashboards only |
| Employee Self-Service | Free | None | None | Own payslips, leave, timesheet only |
| Accountant Partner | Wholesale | Journals if granted by client | Journal posting if granted | Strict client isolation. No system config. |

### Four-Eyes Enforced On
- Payment run: prepare ≠ authorise
- Payroll run: prepare ≠ approve
- Manual journal: submit ≠ approve
- Journal void: request ≠ confirm (2 Administrators)
- Prior period adjustment: submit ≠ approve (must be Administrator)

---

## 18. Journal Types -- Quick Reference

| Type | Auto-Post? | Approval | Period Override? | Key Rule |
|---|---|---|---|---|
| MANUAL | No | Supervisor | No | Cannot self-approve |
| RECURRING | No | Supervisor (each instance) | No | Never auto-posts. Queues for approval. |
| REVERSING | No | Supervisor | No | Auto-generated on reversal date. Must approve. |
| SYSTEM | Yes | None (Audit agent reviews) | No | Agent-generated. Logged as events. |
| OPENING | No | Administrator | Yes -- any period | Migration only. Flagged if after go-live. |
| ADJUSTMENT | No | Administrator | Yes -- locked periods | Prior period only. Written justification required. |
| INTERCOMPANY | N/A | N/A | N/A | Phase 2. Schema present, not in Phase 1 UI. |

---

## 19. Financial Account Types -- Sub-Ledger Reference

| Account Type | Control Account | Normal Balance | Notes |
|---|---|---|---|
| Bank Account | 11010-11014 | Debit | Asset. One FA card per physical account. |
| Term Deposit < 3 months | 11020 | Debit | Cash equivalent. |
| Term Deposit > 3 months | 18050 | Debit | Investment. |
| Credit Card | 24020 | Credit | Liability. Sign convention inverted vs bank. |
| Bank Overdraft | 24010 | Credit | Liability. |
| Short-Term Loan | 24030 | Credit | Liability. |
| Business Loan (long-term) | 26010 | Credit | Liability. Amortisation schedule auto-generated. |
| Director Loan | 26020 / 24042 | Credit (usually) | Liability or Asset. Direction field on card. |
| Finance Lease Liability | 26030 / 24041 | Credit | AASB 16. Linked to ROU asset. |

### Loan Amortisation Formula (P&I)
```
Payment = P × (r(1+r)^n) / ((1+r)^n - 1)
P = principal, r = monthly rate, n = number of payments
```

---

## 20. BAS Labels -- Quick Reference

| Label | Description | Formula |
|---|---|---|
| G1 | Total sales (incl GST) | Sum all sales lines |
| G6 | Sales subject to GST | G1 - G5 (exports + GST-free + input-taxed) |
| **1A** | **GST on sales** | **G8 / 11** |
| G11 | Non-capital purchases (incl GST) | All expense purchases with GST |
| G18 | Total purchases after adjustments | G16 + G17 |
| **1B** | **GST credits (ITC)** | **G18 / 11** |
| **Net GST** | **Amount payable / refundable** | **1A - 1B** |
| W1 | Total wages and salaries | Gross wages from payroll |
| W2 | PAYG withheld from wages | Tax withheld |
| **W5 / Label 4** | **Total PAYG payable** | **W2 + W3 + W4** |

---

## 21. Compliance Deadlines -- Critical Dates (AU)

| Obligation | Frequency | Due Date | Consequence if Missed |
|---|---|---|---|
| STP2 payroll event | Per pay run | Within 1 business day of payday | ATO penalty + interest |
| STP2 year-end finalisation | Annual | 14 July | Employee tax returns delayed |
| BAS -- quarterly | Quarterly | 28th of month after quarter end | ATO penalty + GIC interest |
| Superannuation | Quarterly | 28 days after quarter end | SG Charge (non-deductible) |
| TPAR | Annual | 28 August | $330-$1,650 ATO penalty |
| FBT return | Annual | 21 May (25 Jun via agent) | ATO penalty + FBT liability |
| Award rate update | Annual | 1 July | Below-minimum wages |
| Super rate change | As legislated | 1 July if applicable | Under-contribution |

**Current super rate: 12.0% (from 1 July 2025)**

---

## 22. Sub-Ledger Reconciliation -- Blocking Rules

The following sub-ledger reconciliations are **blocking** (period cannot soft-close until clear):

| Sub-Ledger | Control Account | Tolerance |
|---|---|---|
| Bank accounts (each) | 11010-11014 | $0.00 |
| Accounts Receivable | 12010 | $0.01 |
| Accounts Payable | 21010 | $0.01 |
| Loans (each facility) | 26010-26022 | $1.00 |
| Fixed assets | 16000-17999 | $0.01 |
| Inventory | 13010-13017 | $0.01 |

---

## 23. API Conventions -- Quick Reference

- Base URL: `/api/v1/{resource}`
- Auth: Bearer JWT (15-min access token, 8-hr refresh token with rotation)
- Tenant isolation: `tenant_id` in JWT claims only -- never in URL or body
- Monetary values: **integer cents** -- never floats (500 = $5.00)
- Dates: ISO 8601 UTC (`2025-03-14T09:30:00Z`)
- Soft delete: `DELETE` sets `is_deleted=TRUE` -- never physical delete
- Idempotency: `X-Idempotency-Key` header on all mutations
- Rate limit: 1,000 requests/minute per tenant (HTTP 429 on breach)
- Error format: `{"errors": [{"code": "V1001", "message": "...", "field": "abn"}]}`
- Pagination: cursor-based (`?cursor=abc&limit=25`) -- no offset pagination

---

## 24. Error Code Reference

| Class | Code Range | Severity | Recovery |
|---|---|---|---|
| VALIDATION | V1000-V1999 | Low | User corrects input |
| BUSINESS_RULE | B2000-B2999 | Medium | User resolves rule conflict |
| INTEGRATION | I3000-I3999 | High | Retry / reconnect / workaround |
| SYSTEM | S4000-S4999 | Critical | Auto-retry + admin alert |

---

*End of embedded schema reference. For full detail refer to Toby_Default_COA_v3.0.xlsx and Toby_Universal_COA_v1.0.xlsx.*
