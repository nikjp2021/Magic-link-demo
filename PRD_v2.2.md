# PRODUCT REQUIREMENTS DOCUMENT (PRD)

**Project:** Vestie Mobile Application & Operational Command Center
**Version:** 2.2 — Magic Link Architecture & Interactive Prototype
**Author:** Nikhil Tiwari
**Date:** 22/05/2026

---

## Sign-Off

| Role | Name | Status | Date |
|------|------|--------|------|
| Product Owner (Client) | [Client Name] | *Awaiting Review* | **/2026 |
| Product Manager | Nikhil Tiwari | Approved | 22/05/2026 |
| Development Lead | Radial Code Team | *Awaiting Engineering Review* | **/2026 |

---

## 1. Magic Link System (Non-Technical)

### 1.1 What Is a Magic Link?

Vestie Partners (Mortgage Brokers, Property Lawyers, Building Inspectors, etc.) are busy external professionals. They will not download a custom app, register an account, or remember a password just to update one customer file.

**Solution:** A Magic Link System. When a user pair hits a milestone, the system fires an automated email containing a secure one-click link. The partner taps it, completes a multi-field form in their mobile browser, and the Strapi database updates instantly. No login, no friction.

### 1.2 Lifecycle Flow

```
[1] User Pair hits a property path milestone in the mobile app
         |
[2] System detects milestone → fires automated magic link email to the assigned partner
         |
[3] Partner taps link → smart webform loads (role + stage detected via URL)
         |
[4] Partner completes dynamic fields (yes/no, text, numbers, dates, checkboxes, selections) + uploads PDF proof
         |
[5] Strapi records all field data → Property Path advances to next milestone
```

### 1.3 Cost-Saving Principle

One template. Twelve roles. Zero duplicate screens.

The smart webform reads `?role=xxx&stage=n` from the URL and dynamically renders role-specific labels, field types, options, and file upload targets. This eliminates 11 separate page builds and 33 individual stage screens.

---

## 2. The 12-Role Dynamic Template Matrix

| # | Partner | URL Param | Form Title | Stage 1 | Stage 2 | Stage 3 |
|---|---------|-----------|------------|---------|---------|---------|
| 1 | Mortgage Broker | `broker` | Verify Loan Status | Pre-Approval Submitted | Loan Approved | Funded |
| 2 | Property Lawyer | `lawyer` | Verify Contract Review | Contract Under Review | Exchanged & Signed | Settlement Completed |
| 3 | Building Inspector | `inspector` | Verify Asset Condition | Inspection Scheduled | Report Generated | Issues Resolved |
| 4 | Tax Specialist | `tax` | Verify Depreciation Setup | Depreciation Setup | Ledger Finalized | Submitted to ATO |
| 5 | Real Estate Agent | `agent` | Verify Property Listing | Listing Active | Offer Accepted | Under Contract |
| 6 | Property Valuer | `valuer` | Verify Valuation Report | Valuation Ordered | Site Visit Done | Report Complete |
| 7 | Land Surveyor | `surveyor` | Verify Survey Completion | Survey Ordered | Boundary Marked | Plan Lodged |
| 8 | Strata Manager | `strata` | Verify Strata Clearance | Records Requested | Certificate Issued | Clearance Given |
| 9 | Financial Planner | `finplanner` | Verify Financial Strategy | Strategy Drafted | Approved by Client | Implemented |
| 10 | Insurance Broker | `insurance` | Verify Insurance Cover | Quote Requested | Policy Issued | Policy Bound |
| 11 | Settlement Agent | `settlement` | Verify Settlement Progress | File Opened | Documents Prepared | Settled |
| 12 | Licensed Conveyancer | `conveyancer` | Verify Conveyancing Progress | Search Complete | Section 32 Reviewed | Ready to Exchange |

---

## 3. Enhanced Form Field Types

Each stage of each partner has a custom set of form fields chosen from 8 supported types:

### 3.1 Field Type Reference

| Type | UI Component | Data Type | Example |
|------|-------------|-----------|---------|
| `yesno` | Yes / No toggle pills | `string` (`"yes"` / `"no"`) | "Pre-approval received?" |
| `text` | Single-line text input | `string` | Loan reference number |
| `textarea` | Multi-line text area | `string` | Inspection notes |
| `number` | Numeric input (optional `$` prefix) | `number` | Approved amount |
| `date` | Date picker | `string` (ISO date) | Settlement date |
| `select` | Dropdown menu | `string` | Inspection type |
| `radio` | Radio button group | `string` | Risk profile |
| `checkbox` | Multi-select checkboxes | `string[]` | Conditions satisfied |

### 3.2 Per-Role Field Specifications

#### Mortgage Broker
| Stage | Fields |
|-------|--------|
| Stage 1 | yesno(pre-approval) + text(loan ref) + number(property value, $) + date(expiry) |
| Stage 2 | yesno(full approval) + text(cert ID) + checkbox(conditions met: valuation, income, title, LVR) |
| Stage 3 | number(funded amount, $) + date(settlement) + yesno(disbursed) + textarea(notes) |

#### Property Lawyer
| Stage | Fields |
|-------|--------|
| Stage 1 | yesno(contract received) + text(matter ref) + date(received) |
| Stage 2 | yesno(exchanged) + date(exchange) + textarea(special conditions) |
| Stage 3 | yesno(settled) + date(settlement) + number(amount, $) + text(callback ref) |

#### Building Inspector
| Stage | Fields |
|-------|--------|
| Stage 1 | date(inspection) + select(type: building/pest/both/pre-purchase) + text(booking ref) |
| Stage 2 | radio(outcome: clear/minor/major/uninsurable) + checkbox(issues: structural, plumbing, etc.) + textarea(summary) |
| Stage 3 | yesno(issues resolved) + date(clearance) + text(cert ref) |

#### Tax Specialist
| Stage | Fields |
|-------|--------|
| Stage 1 | yesno(schedule prepared) + text(financial year) + number(est claim, $) |
| Stage 2 | yesno(ledger finalized) + number(total depreciation, $) + textarea(asset summary) |
| Stage 3 | yesno(submitted to ATO) + date(submission) + text(ATO ref) + checkbox(docs included: schedule, register, invoice, cover) |

#### Real Estate Agent
| Stage | Fields |
|-------|--------|
| Stage 1 | text(address) + select(listing type: exclusive/open/auction/private/tender) + date(listing) |
| Stage 2 | yesno(offer accepted) + text(buyer name) + number(offer amount, $) + select(offer status: unconditional/subject to finance/etc.) |
| Stage 3 | yesno(deposit received) + date(contract) + number(deposit amount, $) + textarea(agent notes) |

#### Property Valuer
| Stage | Fields |
|-------|--------|
| Stage 1 | select(purpose: mortgage/pre-purchase/insurance/etc.) + date(order) + text(order ref) |
| Stage 2 | date(site visit) + textarea(observations) + yesno(internal access) |
| Stage 3 | number(valuation amount, $) + date(report) + text(report ref) + yesno(delivered) |

#### Land Surveyor
| Stage | Fields |
|-------|--------|
| Stage 1 | select(type: boundary/topo/strata/identification/contour) + text(instruction ref) + date(order) |
| Stage 2 | yesno(boundaries marked) + date(marking) + number(lots identified) |
| Stage 3 | yesno(plan lodged) + date(lodgement) + text(lodgement ref) |

#### Strata Manager
| Stage | Fields |
|-------|--------|
| Stage 1 | checkbox(docs: roll/by-laws/AGM mins/sinking fund/insurance/financials) + date(request) + text(request ref) |
| Stage 2 | yesno(certificate issued) + date(issue) + number(outstanding levies, $) |
| Stage 3 | yesno(clearance given) + date(clearance) + textarea(strata notes) |

#### Financial Planner
| Stage | Fields |
|-------|--------|
| Stage 1 | checkbox(components: budget/debt/investment/super/insurance/estate) + select(risk profile: conservative to growth) + text(engagement ref) |
| Stage 2 | yesno(approved) + date(approval) + textarea(action items) |
| Stage 3 | yesno(implemented) + date(implementation) + number(funds invested, $) + text(product refs) |

#### Insurance Broker
| Stage | Fields |
|-------|--------|
| Stage 1 | select(type: home & contents/landlord/liability/indemnity/warranty) + text(client ref) + date(quote) |
| Stage 2 | yesno(policy issued) + text(policy number) + number(premium, $) + date(effective) |
| Stage 3 | yesno(bound) + date(binding) + text(binder ref) + checkbox(covers: building/contents/liability/rent/theft/accidental) |

#### Settlement Agent
| Stage | Fields |
|-------|--------|
| Stage 1 | yesno(file opened) + text(file ref) + date(opening) |
| Stage 2 | checkbox(docs: settlement statement/discharge/transfer/adjustments/notice) + textarea(notes) + yesno(docs signed) |
| Stage 3 | number(final amount, $) + date(settlement) + text(settlement ref) + yesno(parties paid) |

#### Licensed Conveyancer
| Stage | Fields |
|-------|--------|
| Stage 1 | checkbox(searches: title/rates/land tax/planning/contaminated/road access) + yesno(title clear) + date(search) |
| Stage 2 | yesno(s32 reviewed) + textarea(client queries) + select(recommendation: proceed/conditions/negotiate/withdraw) |
| Stage 3 | yesno(ready to exchange) + date(exchange ready) + text(exchange ref) |

### 3.3 Property Path Trigger Map

Each Partner is assigned to a specific phase of the user's property purchase journey. The magic link email fires automatically when the user's Property Path reaches a matching milestone. The table below maps each partner role to the trigger event, the property phase they operate in, and which real-world action advances each stage.

| Partner | Trigger Event (Milestone Hit) | Property Phase | Stage 1 Fires When | Stage 2 Fires When | Stage 3 Fires When |
|---------|------------------------------|----------------|-------------------|-------------------|-------------------|
| Mortgage Broker | User taps "Finance" step | Finance | Pre-approval requested in app | Bank issues conditional approval | Settlement date confirmed by solicitor |
| Property Lawyer | User reaches "Legal Review" step | Legal & Settlement | Vendor solicitor drafts contract | User instructs lawyer to exchange | Settlement date confirmed |
| Building Inspector | Offer accepted → Due Diligence opens | Due Diligence | User books inspection via app | Inspector completes site visit | 7-day review window opens |
| Tax Specialist | User enters "Tax" step | Due Diligence | Financial year end approaching | Mid-year depreciation review | ATO lodgement deadline |
| Real Estate Agent | User marks "List Property" | Acquisition | Property goes live on market | Verbal offer accepted | Contract signed by vendor |
| Property Valuer | Lender requests valuation in app | Due Diligence | Valuation ordered via lender portal | Property access granted by owner | Report due to lender |
| Land Surveyor | Subdivision flagged during Due Diligence | Due Diligence | Survey instruction issued | Site marking completed | Council lodgement deadline |
| Strata Manager | Property identified as strata titled | Due Diligence | Strata records requested by conveyancer | Certificate required for exchange | Clearance needed before settlement |
| Financial Planner | User selects "Financial Plan" from menu | Finance | Initial consultation booked with planner | Client reviews strategy draft | Implementation date set |
| Insurance Broker | Property purchase or refinance triggered | Finance | User requests quote from app | Policy issued by insurer | Settlement requires binding insurance |
| Settlement Agent | Contract goes unconditional | Legal & Settlement | File opened after cooling-off expiry | Documents prepared T-7 days before settlement | Settlement day (T) |
| Licensed Conveyancer | User reaches "Conveyancing" step | Legal & Settlement | Property identified for purchase | Section 32 received from vendor | Exchange date confirmed |

#### Trigger Flow by Property Phase

```
Phase 1: Acquisition
  Real Estate Agent
    Stage 1: Property listed on market
    Stage 2: Offer received and accepted
    Stage 3: Contract signed → moves to Legal Phase

Phase 2: Finance
  Mortgage Broker
    Stage 1: Pre-approval requested
    Stage 2: Full approval granted
    Stage 3: Funds disbursed at settlement
  Financial Planner
    Stage 1: Initial consultation booked
    Stage 2: Strategy approved by client
    Stage 3: Plan implemented
  Insurance Broker
    Stage 1: Quote requested
    Stage 2: Policy issued
    Stage 3: Policy bound for settlement

Phase 3: Due Diligence
  Building Inspector
    Stage 1: Inspection booked
    Stage 2: Report generated with findings
    Stage 3: Issues resolved or waived
  Property Valuer
    Stage 1: Valuation ordered
    Stage 2: Site visit completed
    Stage 3: Report delivered to lender
  Land Surveyor
    Stage 1: Survey instructed
    Stage 2: Boundaries marked on site
    Stage 3: Plan lodged with council
  Tax Specialist
    Stage 1: Depreciation schedule prepared
    Stage 2: Ledger finalized
    Stage 3: Submitted to ATO
  Strata Manager
    Stage 1: Records requested
    Stage 2: Certificate issued
    Stage 3: Clearance given

Phase 4: Legal & Settlement
  Property Lawyer
    Stage 1: Contract received and reviewed
    Stage 2: Contracts exchanged
    Stage 3: Settlement completed
  Licensed Conveyancer
    Stage 1: Searches complete and clear
    Stage 2: Section 32 reviewed
    Stage 3: Ready to exchange
  Settlement Agent
    Stage 1: File opened
    Stage 2: Documents prepared
    Stage 3: Settled — all parties paid
```

---

## 4. Interactive Prototype

### 4.1 File Structure

```
D:\Vestie\
  index.html      — Command Center dashboard (role grid, vault sim, audit log)
  demo.html       — Universal mobile form (all 12 roles, 36 stage views, 8 field types)
  PRD_v2.2.md     — This document
```

### 4.2 How to Run

1. Open `index.html` in any browser — launches the Command Center
2. Click any partner tile — opens `demo.html?role=<id>&stage=1` in a new tab
3. Use the Demo Controller panel to switch roles and cycle through stages
4. Fill in the dynamic form fields and submit to see the Strapi payload

### 4.3 URL Scheme

```
demo.html?role=broker&stage=1       → Mortgage Broker, Stage 1
demo.html?role=inspector&stage=2    → Building Inspector, Stage 2
demo.html?role=conveyancer&stage=3  → Licensed Conveyancer, Stage 3
```

### 4.4 Key Demo Features

- **12 role tiles** in Command Center with colour-coded category tags
- **3-stage stepper** per role with dot indicators
- **8 field types** — yes/no pills, text, textarea, number ($), date, dropdown, radio groups, checkbox groups
- **Per-stage field sets** — each of the 36 stage views has a unique field layout
- **Inline validation** — required fields show red error messages on submit
- **File upload** simulation with document staging
- **Form data payload** — submit collects all field values into a JSON payload shown on the success screen and logged to the terminal
- **Secure token countdown** with expiration and HTTP 403 simulation
- **Real-time terminal** logging all events as Strapi would

---

## 5. Technical Notes for Radial Code

### 5.1 Component Architecture

```
Magic Link Template
  ├── Header (brand + role tag)
  ├── Form Card
  │   ├── Title (dynamic via role data)
  │   ├── Stage Pill Selector (3 milestones)
  │   ├── Dynamic Field Set (rendered per stageFields[n])
  │   │   ├── Yes/No toggle pills
  │   │   ├── Text inputs / Textareas
  │   │   ├── Number inputs with optional $ prefix
  │   │   ├── Date pickers
  │   │   ├── Select dropdowns
  │   │   ├── Radio button groups
  │   │   └── Checkbox groups
  │   ├── File Upload Zone (single PDF, 10MB max)
  │   └── Submit Button (validates all required fields)
  └── Footer (security badge)
```

### 5.2 Strapi Data Model Mapping

| Strapi Collection | Field | Source |
|-------------------|-------|--------|
| `user_pairs` | `id` | System |
| `partner_verifications` | `role_type` | URL param |
| `partner_verifications` | `milestone_stage` | Current stage index |
| `partner_verifications` | `stage_name` | Stage label |
| `partner_verifications` | `form_data` | Key-value pairs from all field types |
| `partner_verifications` | `document_url` | Uploaded file path |
| `partner_verifications` | `status` | Submitted / Complete |
| `property_paths` | `current_stage` | Updated on verification completion |

The `form_data` field stores a JSON object containing every field value submitted by the partner. For example:

```json
{
  "preApproved": "yes",
  "loanRef": "LON-2026-0042",
  "estValue": 850000,
  "expiryDate": "2026-08-15",
  "conditions": ["Valuation met", "Title clear"]
}
```

### 5.3 Validation Logic

| Field Type | Required Rule | Error Message |
|-----------|--------------|---------------|
| `yesno` | One of Yes/No must be selected | "This field is required" |
| `text` | Must not be empty | "This field is required" |
| `textarea` | Must not be empty (if required) | "This field is required" |
| `number` | Must have a valid numeric value | "This field is required" |
| `date` | Must have a date selected | "This field is required" |
| `select` | Must not be the default placeholder | "This field is required" |
| `radio` | One option must be selected | "This field is required" |
| `checkbox` | At least one must be checked | "This field is required" |

File upload is always required. Submit is blocked until all required fields pass validation.

### 5.4 Security Implementation

- **Pre-signed URLs with 14-day validity** — magic link remains accessible for the partner's response window
- Single-use token invalidated after first successful submission
- HTTP 403 returned if link is accessed after expiry or after submission
- All uploads validated to PDF format, max 10MB
- Audit trail logged to Strapi `verification_logs`
- Expired links can be re-issued by the admin via the Command Center

---

## 6. Revision History

| Date | Version | Changes |
|------|---------|---------|
| 22/05/2026 | 2.2.0 | Full 12-role matrix, interactive HTML prototype, Command Center, Magic Link specification. Enhanced forms with 8 field types across 36 stage views. |
| 22/05/2026 | 2.2.1 | Added Property Path Trigger Map (Section 3.3) mapping each partner form to its property milestone trigger event. Updated lifecycle flow. |
