# Product Requirements Document

**Project:** Vestie — Property Co-Buying Platform
**Author:** Nikhil Tiwari
**Latest Version:** 2.6 (23/05/2026)

## 1. Overview

Vestie connects co-buyers who purchase property together. The platform has two operational sides:

- **Partner-facing**: External professionals (brokers, lawyers, inspectors) receive secure Magic Links via email. They tap the link, fill a form in their mobile browser, and the system records their verification — no app install or login needed.
- **Admin-facing**: Anna (the System Admin) monitors traffic, tracks revenue, configures partner fees, and manages invoicing from a single dashboard.

Revenue comes from two streams: **User-Facing 50/50 Cost Splits** (paid in-app via Apple Pay / Google Pay / Card) and **Partner-Facing Commission Ledger** (calculated by the system, billed manually by Anna via Xero).

## 2. Magic Link System

### 2.1 How It Works

One reusable form template serves all 12 partner roles. The URL determines which role and stage to show: `demo.html?role=broker&stage=1`. No duplicate screens.

**Flow:**
1. A co-buyer pair hits a milestone on their Property Path
2. The system emails a Magic Link to the assigned partner
3. The partner taps the link, sees a form tailored to their role and current stage
4. They fill in fields, upload a PDF (10MB max), and submit
5. Strapi records the data and the Property Path advances

### 2.2 The 12 Roles

| # | Partner | Phase | Stage 1 | Stage 2 | Stage 3 |
|---|---------|-------|---------|---------|---------|
| 1 | Mortgage Broker | Finance | Pre-Approval | Loan Approved | Funded |
| 2 | Property Lawyer | Legal & Settlement | Contract Review | Exchanged | Settlement |
| 3 | Building Inspector | Due Diligence | Inspection Scheduled | Report Generated | Issues Resolved |
| 4 | Tax Specialist | Due Diligence | Depreciation Setup | Ledger Finalized | Submitted to ATO |
| 5 | Real Estate Agent | Acquisition | Listing Active | Offer Accepted | Under Contract |
| 6 | Property Valuer | Due Diligence | Valuation Ordered | Site Visit Done | Report Complete |
| 7 | Land Surveyor | Due Diligence | Survey Ordered | Boundary Marked | Plan Lodged |
| 8 | Strata Manager | Due Diligence | Records Requested | Certificate Issued | Clearance Given |
| 9 | Financial Planner | Finance | Strategy Drafted | Approved by Client | Implemented |
| 10 | Insurance Broker | Finance | Quote Requested | Policy Issued | Policy Bound |
| 11 | Settlement Agent | Legal & Settlement | File Opened | Documents Prepared | Settled |
| 12 | Licensed Conveyancer | Legal & Settlement | Search Complete | Section 32 Reviewed | Ready to Exchange |

### 2.3 Field Types (8 Supported)

`yesno` (toggle pills), `text`, `textarea`, `number` (with optional $), `date`, `select` (dropdown), `radio`, `checkbox` (multi-select).

Each role's 3 stages have unique field combinations — see `demo.html` for the full 36-view layout.

### 2.4 Security

- Magic Links have **7-day validity** (pre-signed URL with expiry)
- Single-use token — invalidated after first successful submission
- Expired or consumed links return **HTTP 403**
- All uploads validated to PDF format, 10MB max
- Admin can re-issue expired links from the Command Center

## 3. Admin Dashboard

Anna's dashboard (`admin.html`) has 5 modules:

| Module | What It Shows | Source |
|--------|--------------|--------|
| **A — App Traffic** | Total downloads, live active users, compliance-verified profiles | System API ping counters |
| **B — Revenue Ledger** | Fees earned (paid), outstanding fees, partner payables owed (15% kickback) | Append-only `referral_records` |
| **C — Subscriptions** | Free tier count, premium subscribers, monthly recurring revenue (MRR) | `core_user_profiles` tier field |
| **D — Stagnation Alerts** | Red badge on deals inactive for 7+ days | Date diff on `status_history` |
| **E — CSV Export** | One-click download of all transaction rows | Client-side table-to-CSV |

**Sidebar Simulators:** Buttons to bump download count (+1), simulate concurrent traffic (+random 5–29), convert free user to premium (+$9.90 MRR), and process a co-buyer payment.

## 4. Partner Fee Configuration

Hidden fields on the `vestie_partners` Strapi collection (invisible to the public app) control automatic fee calculation.

### 4.1 Hidden Fields

| Field | Type | Purpose |
|-------|------|---------|
| `billing_type` | Enum | Flat Fee or Percentage |
| `base_fee_amount` | Decimal | Flat fee in AUD (e.g. $120 for Tax Specialist) |
| `commission_rate` | Decimal | Percentage as decimal (e.g. 0.015 = 1.5%) |
| `trigger_milestone` | Enum | Event that fires the fee (e.g. "Report Delivered") |

### 4.2 Calculation Rules

- **Flat Fee:** `Invoice = base_fee_amount` — used for fixed-price services (Tax $120, Lawyer $300, Inspector $180)
- **Percentage:** `Invoice = Property Price x LVR x commission_rate x Vestie Split (20%)` — used for variable services (Broker 1.5% → ~$1,800 on $750k property)

### 4.3 Trigger Milestones

| Milestone | Roles |
|-----------|-------|
| Report Delivered | Tax Specialist, Building Inspector, Valuer, Surveyor |
| Engagement Confirmed | Lawyer, Conveyancer, Settlement Agent |
| Loan Settlement | Mortgage Broker, Financial Planner |
| Pre-Approval | Mortgage Broker (alternate) |
| Policy Bound | Insurance Broker |
| Strategy Implemented | Financial Planner |

The Magic Link handler reads these fields at submission time and writes the calculated invoice row to `invoice_records`.

## 5. Invoice Generation & Templates

Two streams, two templates.

### 5.1 Stream 1: User-Facing Shared Cost Split (Template A)

Rendered as a mobile card on the Property Path. Shows the total cost, applies a 50/50 split, and presents a payment button for the user's share.

**Template A layout:**
```
Property: 123 Main Street, Sydney
Total Cost: $600.00
Split: 50/50
Your Share: $300.00
[Pay with Apple Pay] [Google Pay] [Credit Card]
```

Once paid, the `invoice_records` row flips to `Paid` for that user. The other user's payment is tracked independently.

### 5.2 Stream 2: Partner Commission Statement (Template B)

Rendered in the admin dashboard. Auto-generated when a partner hits their `trigger_milestone` via Magic Link. Pulls the partner's hidden profile fields to compute the amount, then locks a static row.

**Template B layout:**
```
Partner: Hometown Mortgages
Commission: $1,800.00
Invoice: INV-A7F3D9-0001
Status: PENDING
[Verify & Lock Invoice]
```

Once verified, the row is frozen — Anna exports it to Xero at end of month. No automated B2B payment gateway in MVP scope.

### 5.3 How They Share `invoice_records`

Both streams write to the same `invoice_records` collection. The `invoice_type` field distinguishes them (`User_Split` vs `Partner_Commission`).

## 6. Post-Payment Flow

When a co-buyer pays their 50% share, the system triggers a 3-step cycle automatically — no extra builds needed.

### 6.1 The 3-Step Cycle

```
Payment Gateway Success Callback
        │
        ▼
┌───────────────────┐
│ Step 1: Paystub   │  The checkout card morphs into an itemised receipt showing the paid amount,
│ (On-Screen)       │  transaction reference ID, and co-buyer contribution status — all within the
│                   │  same screen slot.
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Step 2: Email     │  A professional confirmation email fires to the user's inbox containing
│ (Automated)       │  a downloadable PDF copy of the exact paystub.
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│ Step 3: Hub       │  The paystub is filed inside a new "My Finances" section in the user's
│ (Financial Link)  │  profile. Users can revisit past transactions anytime.
└───────────────────┘
```

All three steps use the **Unified Dynamic Renderer Component** — no standalone layouts.

### 6.2 Multi-Scenario Paystub Framework

The same paystub and Financial Hub templates support 3 real-world scenarios by pulling data from Strapi fields at render time:

| # | Scenario | Example | Calculation | Receivable Type |
|---|----------|---------|-------------|-----------------|
| 1 | User Legal Fee Split | Co-Ownership Agreement Setup ($600) | $600 ÷ 2 = $300/user | User cost split |
| 2 | User Partner Service Split | Building & Pest Inspection ($240) | $240 ÷ 2 = $120/user | User cost split |
| 3 | B2B Partner Commission | Broker corporate commission ($1,200) | Property Price × LVR × Rate × 20% | Anna's receivable ledger |

**How it works:** When a payment clears or a partner hits a milestone, the post-payment engine reads the `asset_description` and `billing_stream` from the `invoice_records` row, picks the matching scenario labels, and renders the universal paystub template with the correct text and amounts. No custom code per scenario.

### 6.3 Admin Receivables Tracker

Anna's dashboard now includes a dedicated receivables ledger that aggregates all outstanding B2B commission rows (`billing_stream = B2B_Partner_Receivable`, `ledger_clear_status = Unpaid_Outstanding`). Rows flash a warning flag if unpaid for over 7 days. Anna marks them as settled when the money clears her bank account.

### 6.4 `user_paystubs` Schema

| Field | Type | Notes |
|-------|------|-------|
| `paystub_uuid` | UUID | Primary key |
| `linked_invoice_id` | Relation | Links to `invoice_records` |
| `clear_timestamp` | DateTime | Exact payment time |
| `transaction_ref` | String | Bank/Stripe processor code |
| `vault_file_path` | Media | Private PDF link |

### 6.5 Use Cases

| # | Module | Screen | Role | Description |
|---|--------|--------|------|-------------|
| 7.1 | FinTech | Post-Payment Paystub | Mobile User | After payment clears, show itemised receipt with asset details, pricing, split, bank transaction number, and green verification stamp |
| 7.2 | Communication | Automated Receipt Email | System Engine | Background transactional email bundling receipt variables sent to user's verified inbox |
| 7.3 | FinTech | Financial Link Directory | Mobile User | "My Finances" tab in profile menu listing all past transactions and billing statements |
| 8.1 | FinTech | Multi-Use Case Split Card | Mobile User | When any shared expense milestone unlocks, the app loads the template card dynamically populated with the correct text and price from Strapi tables |
| 8.2 | FinTech | On-Screen Dynamic Paystub | Mobile User | When a payment clears, the checkout screen transitions to an itemised receipt with transaction codes and green validation stamp |
| 8.3 | FinTech | Financial Link Folder | Mobile User | "My Finances" lists all historically paid splits; tapping any row expands its original paystub view |
| 8.4 | FinTech | Admin Receivables Tracker | System Admin | Ledger on Anna's dashboard aggregating all outstanding corporate commissions owed to Vestie |

## 7. Strapi Schema Reference

### 6.1 `vestie_partners`

| Field | Type | Private | Notes |
|-------|------|---------|-------|
| `full_name` | Text | No | Public |
| `professional_role` | Text | No | Public |
| `avatar_image` | Media | No | Public |
| `provider_bio` | Text | No | Public |
| `billing_type` | Enum | Yes | Flat or Percent |
| `base_fee_amount` | Decimal | Yes | AUD |
| `commission_rate` | Decimal | Yes | e.g. 0.015 |
| `trigger_milestone` | Enum | Yes | Event that fires fee |
| `plan_tier` | Enum | No | Free, Starter, Pro, Enterprise |
| `renewal_date` | Date | No | Subscription renewal |
| `subscription_status` | Enum | No | Active, Expiring, Expired, Suspended |

### 6.2 `core_user_profiles`

| Field | Type | Notes |
|-------|------|-------|
| `user_uuid` | UUID | Primary key |
| `compliance_signed` | Boolean | Default false |
| `compliance_timestamp` | DateTime | Nullable |
| `subscription_tier` | Enum | Free or Premium |
| `monthly_billing_rate` | Decimal | AUD |

### 6.3 `vestie_referrals`

| Field | Type | Notes |
|-------|------|-------|
| `current_stage` | Integer | 1, 2, or 3 |
| `days_inactive` | Integer | Days since last activity |
| `risk_level` | Enum | Low, Medium, High |
| `assigned_partner` | Relation | Many-to-one to `vestie_partners` |

### 6.4 `partner_verifications`

| Field | Type | Notes |
|-------|------|-------|
| `role_type` | String | URL param |
| `milestone_stage` | Integer | Current stage index |
| `stage_name` | String | Stage label |
| `form_data` | JSON | Key-value field values |
| `document_url` | String | Uploaded PDF path |
| `status` | Enum | Submitted or Complete |

### 7.5 `invoice_records`

| Field | Type | Notes |
|-------|------|-------|
| `invoice_uuid` | UUID | Primary key |
| `parent_referral_id` | Relation | Links to `referral_records` |
| `invoice_type` | Enum | User_Split or Partner_Commission |
| `billing_stream` | Enum | User_Cost_Split or B2B_Partner_Receivable |
| `asset_description` | String | e.g. "Building & Pest Inspection Split" |
| `gross_amount` | Decimal | AUD |
| `individual_share` | Decimal | AUD (50% for user splits) |
| `accounting_receivable` | Decimal | Exact balance due to Vestie |
| `invoice_status` | Enum | Unpaid or Paid |
| `ledger_clear_status` | Enum | Unpaid_Outstanding or Paid_Settled |

### 7.6 `invoice_templates`

| Field | Type | Notes |
|-------|------|-------|
| `template_id` | UUID | Primary key |
| `template_type` | Enum | User_Split_Card or Partner_Commission_Statement |
| `split_ratio` | Decimal | Default 0.5 (50/50) |
| `payment_methods` | JSON | Apple Pay, Google Pay, Card |

### 7.7 `user_paystubs`

| Field | Type | Notes |
|-------|------|-------|
| `paystub_uuid` | UUID | Primary key |
| `linked_invoice_id` | Relation | Links to `invoice_records` |
| `clear_timestamp` | DateTime | Exact payment time |
| `transaction_ref` | String | Bank/Stripe processor code |
| `vault_file_path` | Media | Private PDF link |

## 8. User Stories

### US-ADM-004 (Automated Financial Accounting)
*As Anna, I want to see an auto-calculated ledger (earned, outstanding, reverse fees) so I don't run manual checks.*
- **AC:** When a partner submits their Magic Link at a fee-triggering milestone, the system pulls that partner's profile rules, updates `referral_records`, recalculates balances, and displays the update in <2 seconds.

### US-ADM-005 (Deal Stagnation Alerts)
*As Anna, I want red alert flags on deals inactive for 7+ days so I can follow up with partners.*
- **AC:** A daily background pass checks `status_history` timestamps. If delta >7 days, the record's alert flag flips to true and a red badge appears on the row.

### US-FIN-006 (User Shared Cost Split)
*As a co-buyer, I want to see my 50% split and pay it in-app instead of manual bank transfers.*
- **AC:** When a shared-cost milestone unlocks, the app shows `Total $600 → Your Share $300` with a Pay button. Payment state is isolated per user. On success, the user's row flips to `Paid`.

### US-FIN-007 (Partner Commission Recording)
*As Anna, I want the system to lock a static commission row the moment a partner hits their trigger milestone, so I can export to Xero without recalculation errors.*
- **AC:** On Magic Link submission at trigger milestone, the engine computes the fee, writes a locked `Unpaid` row to `invoice_records`, and displays it on the admin ledger.

### US-INV-008 (User Split Card Template)
*As a co-buyer, I want a clear inline card on my Property Path showing the split amount with one-tap payment options.*
- **AC:** The Template A card shows property address, total cost, 50% split, individual share, and Apple Pay / Google Pay / Credit Card buttons. States: unpaid (buttons active) → paid (success checkmark).

### US-INV-009 (Partner Commission Statement Template)
*As Anna, I want the partner commission displayed as a locked statement row I can verify, so I know exactly who to bill.*
- **AC:** Template B shows partner name, computed commission, invoice UUID, and a Verify & Lock button. After verification, the row freezes and appears on the admin ledger.

### US-FIN-010 (On-Screen Paystub & Email Automation)
*As a co-buyer, I want to see a payment receipt on-screen and receive an email confirmation the moment my payment clears.*
- **AC:** When the banking gateway returns `Success 200`, the checkout screen instantly transitions to an itemised paystub card, and the background email engine fires a confirmation with payment breakdown to the user's verified inbox.

### US-FIN-011 (The Financial Link Repository)
*As a co-buyer, I want a permanent "My Finances" folder in my profile to review all past split invoices and paystubs.*
- **AC:** When the user navigates to their profile and taps "My Finances", the UI queries the `user_paystubs` collection, fetches all settled records linked to their account, and displays them in a scannable history list.

### US-FIN-012 (Dynamic Multi-Scenario Paystub)
*As a co-buyer, I want the paystub receipt to accurately show the service I paid for, whether it's a legal setup fee, an inspection bill, or any other shared cost.*
- **AC:** When the payment gateway returns success, the system transforms the checkout screen into a universal itemised paystub template that dynamically maps the item labels matching the use case (e.g. "Building & Pest Inspection"), records a processor reference ID, and dispatches a verification email.

### US-FIN-013 (Financial Link Directory Management)
*As a co-buyer, I want my "My Finances" folder to list multiple past payment receipts across different service types, and let me tap any row to re-open its original paystub.*
- **AC:** After the user has cleared multiple dynamic split costs (e.g. both a Legal Setup Fee and an Inspection Bill), the Financial Link loads a history list detailing each transaction. Each row expands into its original itemised paystub when tapped.

## 9. Interactive Prototypes

| File | Purpose | How to Open |
|------|---------|-------------|
| `index.html` | Command Center — 12 partner tiles, security vault, audit log | Open directly |
| `demo.html` | Magic Link form — all 12 roles, 3 stages, 8 field types | Click a tile in index.html |
| `admin.html` | Admin Dashboard — Modules A–E, simulators, CSV export | Click Admin link in header |
| `fee_config.html` | Fee Config Engine — hidden Strapi fields driving auto-calculation | Click Fee Config link in header |
| `invoice_demo.html` | Invoice Templates — Template A (Split Card) and Template B (Commission Statement) toggle | Click Invoice Demo tile in index.html |
| `paystub_demo.html` | Unified Receivables & Paystub Engine — 3-scenario toggle (Legal Split / Inspection / Broker Commission), paystub, email dispatch, Financial Hub, and admin receivables ledger | Click Paystub & Hub tile in index.html |

All files run in any browser with zero dependencies.

## 10. Implementation Notes

- **Priority:** Modules A & B first (most business value), then C & D, then E (CSV export).
- **15% Kickback:** Reverse partner fees = 15% of referral fee. Stored as calculated field in `referral_records`, not hardcoded in UI.
- **CSS Theme:** All files use shared CSS variables (`--bg-primary: #0F1117`, `--bg-secondary: #1A1D27`, `--border: #2D3148`, etc.) declared at root level in each file for standalone use.
- **CSV Export:** Client-side JavaScript reads table rows, formats as CSV, triggers download via Blob URL.
- **Simulation:** All data is hardcoded in JavaScript arrays/objects. Sidebar buttons trigger state changes. No backend required.
- **Demo Data:** Default property price = $750,000, LVR = 80% for percentage commission calculations.

## Revision History

| Date | Version | Summary |
|------|---------|---------|
| 22/05/2026 | 2.2 | Magic Link system: 12-role matrix, 36 stage views, 8 field types, Command Center, 7-day security |
| 23/05/2026 | 2.4 | Admin Dashboard (Modules A–E), Hidden Fee Configurations, two-stream Invoice Generation |
| 23/05/2026 | 2.5 | Dynamic Invoice Templates: Template A (User Split Card) and Template B (Commission Statement), `invoice_templates` schema, interactive prototype |
| 23/05/2026 | 2.6 | Post-Payment Flow: 3-step cycle (paystub, email, Financial Hub), `user_paystubs` schema, US-FIN-010/011, interactive prototype |
| 23/05/2026 | 2.6a | Multi-Scenario Paystub Framework: 3 scenarios (Legal Split / Inspection / Broker Commission), expanded `invoice_records` schema (`billing_stream`, `asset_description`, `accounting_receivable`, `ledger_clear_status`), admin receivables tracker, US-FIN-012/013, use cases 8.1–8.4 |
