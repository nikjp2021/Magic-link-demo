# PRODUCT REQUIREMENTS DOCUMENT (PRD) ADDENDUM v2.4

## Admin Operations Dashboard & Complete Revenue Dashboard Engine

| Document | Details |
|----------|---------|
| **Project** | Vestie Mobile Application & Operational Command Center |
| **Version** | 2.4 (Admin Operations & Complete Revenue Dashboard Engine) |
| **Author** | Nikhil Tiwari (Product Manager) |
| **Target Audience** | Anna (Operations Admin) & Radial Code (Dev Team) |

---

## Document Revisions

| Date | Version | Author | Section Changed | Description of Changes |
|------|---------|--------|-----------------|------------------------|
| 23/05/2026 | 2.4.0 | Nikhil Tiwari | All Sections | Engineered complete specifications for app metrics tracking, financial calculation logs, subscription engines, and automated ledger matrices. |

## Sign-Off & Approvals

*By approving this addendum, the Client locks the administrative requirements, real-time dashboard data parameters, and system logging metrics. Radial Code is authorized to configure these fields directly within the existing Strapi backend architecture.*

| Role | Name | Signature / Approval Status | Date |
|------|------|----------------------------|------|
| **Product Owner (Client)** | [Client Name] | *Awaiting Review* | **/**/2026 |
| **Product Manager** | Nikhil Tiwari | Approved | 23/05/2026 |
| **Development Lead** | Radial Code Team | *Awaiting Engineering Review* | **/**/2026 |

---

## Table of Contents

1. [Executive Command Dashboard Layout (Anna's View)](#1-executive-command-dashboard-layout-annas-view)
2. [Component Consolidation & Dynamic System Matrix](#2-component-consolidation--dynamic-system-matrix)
3. [Comprehensive Use Case Specification Grid](#3-comprehensive-use-case-specification-grid)
4. [MVP User Stories & Acceptance Criteria](#4-mvp-user-stories--acceptance-criteria)
5. [Interactive HTML Command Dashboard Engine](#5-interactive-html-command-dashboard-engine)
6. [Strapi Schema Extensions](#6-strapi-schema-extensions)
7. [Implementation Notes for Radial Code](#7-implementation-notes-for-radial-code)
8. [Hidden Partner Fee Configurations (Strapi Core Setup)](#8-hidden-partner-fee-configurations-strapi-core-setup)

---

## 1. Executive Command Dashboard Layout (Anna's View)

To manage the entire real estate marketplace safely and track revenue effortlessly, the Admin Dashboard acts as Anna's **Business Command Center**. Instead of forcing developers to program complex visual graphs from scratch, Radial Code will configure these tracking modules using their existing **Strapi CMS engine infrastructure layout**.

When Anna logs into the backend website, the home command screen presents four clear, automated informational summary blocks:

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        VESTIE MASTER SYSTEM COMMAND CENTER                             │
├────────────────────────────────────────────────────────────────────────────────────────┤
│                                                                                        │
│  📊 MODULE A: APP TRAFFIC & PERFORMANCE                                                 │
│  ├── Total App Store Downloads:  [ 12,450 ]                                            │
│  ├── Live Users Active Right Now: [ 340 ]                                              │
│  └── Compliance Verified Users:  [ 8,920 ] ◄─── (Captured with Signed Timestamps)     │
│                                                                                        │
│  💰 MODULE B: AUTOMATED REFERRAL CASH LEDGER                                           │
│  ├── Total Fees Earned (Paid):   [ $24,500 AUD ]                                       │
│  ├── Outstanding Fees Invoiced:  [ $12,300 AUD ]                                       │
│  └── Reverse Partner Fees Owed:  [ $1,170 AUD ]  ◄─── (15% Kickback Rule Ledger)       │
│                                                                                        │
│  💎 MODULE C: SUBSCRIPTION & SYSTEM MONETIZATION                                       │
│  ├── Total Active Free Tier Users: [ 8,500 ]                                           │
│  ├── Active Premium Subscribers:   [ 420 ]     ◄─── (Future Tier Deployment)           │
│  └── Total Monthly Subscription MRR: [ $4,158 AUD ]                                     │
│                                                                                        │
│  🚀 GLOBAL ADMINISTRATIVE ACTIONS                                                     │
│  └── [ 🧾 EXPORT MASTER TRANSACTION ACCOUNTING CSV ]   [ 🚨 VIEW 7-DAY STAGNATE DEALS ] │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

---

## 2. Component Consolidation & Dynamic System Matrix

The matrix below maps the raw client feedback tracking requirements and transaction metrics into streamlined back-end system calculations.

| Target Analytics Metric | Proposed Implementation Strategy | Strategic Business Rationale (Why) | Code & Data Efficiency | Admin UI/UX Benefit |
|-------------------------|--------------------------------|--------------------------------------|------------------------|----------------------|
| **App Growth Metrics** *(Downloads & Live Users)* | **System API Ping Counter Hooks:** The mobile application tracks app initialization events. Downloads are populated from App Store/Google Play analytics tracking scripts, and concurrent active users are incremented via a 60-second database heartbeat loop. | Gives Anna absolute visibility into user adoption trends and active system load patterns without manual counts. | High execution speed. Uses clean data counters instead of complex monitoring infrastructure. | Real-time traffic visualization showing server stability and marketing reach. |
| **Automated Referral Accounting** *(Receivables & Payables)* | **Relational Stored Revenue Ledgers:** The engine monitors row entries within the append-only `referral_records` database list. Total Earned filters for rows marked `fee_paid == true`, Outstanding filters for `fee_paid == false`, and Reverse Fees aggregate `reverse_referral_amount`. | Eliminates human bookkeeping mistakes and manual accounting updates. Fees are system-locked at the moment of trigger. | Zero manual database query delays. Pre-computes math balances on record creation. | Clean, structured transparency showing exactly who owes Vestie money and what Vestie owes back to partners. |
| **Subscription Invoicing** *(Monetization Tracking)* | **System User Schema Extension Toggles:** Add `subscription_tier` (Enum: Free/Premium), `billing_cycle` (Enum: Monthly/Annual), and `stripe_customer_id` (Text Field) directly onto the core user profile row configuration inside Strapi. | Establishes the infrastructure framework to scale and turn on subscription tiers instantly without rewriting application database layouts post-launch. | Integrates directly into existing authentication tables without slowing database read paths. | Simplifies user monitoring, tracking customer growth rates, and calculating recurring revenue metrics. |
| **Deal Stagnation Tracking** *(7-Day Operational Breaker)* | **Automated Chronological Date Hooks:** A system background rule runs daily to evaluate the difference between the current system date and the latest entry inside a record's `status_history` array. If the delta is > 7 Days, trigger a critical alert row flag. | Protects user experience by automatically highlighting transactions that have stalled in a partner's pipeline. | Low overhead loop. Runs as a lightweight automated database pass once every 24 hours. | Immediate proactive management. Anna sees precisely which external partners require prompt follow-ups. |

---

## 3. Comprehensive Use Case Specification Grid

```
       [Strapi Backend Core: Extension User Table Schema]
 ─────────────────────────────────────────────────────────────
  Collection Identifier: core_user_profiles
 ─────────────────────────────────────────────────────────────
  ├── Field: user_uuid            [DataType: UUID, Constraints: Unique PK]
  ├── Field: compliance_signed    [DataType: Boolean, Default: False]
  ├── Field: compliance_timestamp [DataType: DateTime, Nullable: True]
  ├── Field: subscription_tier    [DataType: Enumeration, Options: Free, Premium]
  └── Field: monthly_billing_rate [DataType: Decimal, Currency: AUD, Default: 0.00]
```

| S No | Module | Use Case / Screen | User Role | Functional Description | Visuals |
|-------|--------|-------------------|-----------|------------------------|---------|
| **4.1** | Analytics | Traffic Operations Center | System Admin | Displays live system usage metrics: Total Platform Downloads, Active Live Users (60-second ping logs), and total Compliance Verified Accounts. | `[Admin Analytics Frame 1]` |
| **4.2** | FinTech | Automated Cash Ledger | System Admin | Aggregates and calculates absolute financial tracking categories: Fees Earned (Settled transactions), Outstanding Fees Invoiced (Pending collections), and Reverse Referral Payables due back to partners. | `[Admin Ledger Frame 2]` |
| **4.3** | Premium | Monetization Monitor | System Admin | Tracks core subscription data arrays: Total active free accounts, active premium tier subscribers, and live running Monthly Recurring Revenue (MRR) formulas. | `[Admin Subscription View]` |
| **4.4** | Operations | Stagnation Alert Matrix | System Admin | Automatically flags deals stuck in the pipeline. If a record has had no updates for more than 7 days, it appends a red exclamation point next to the entry row. | `[Admin Alerts Workspace]` |
| **4.5** | FinTech | One-Click Ledger Export | System Admin | A master button at the top of the workspace. Clicking it dynamically downloads a filtered spreadsheet layout (`.csv`) containing all core matching entries for offline reconciliation. | `[Admin Export Component]` |

---

## 4. MVP User Stories & Acceptance Criteria

### Use Story ID: US-ADM-004 (Automated Financial Accounting Module)

- **As an:** Executive Platform Super-Administrator (Anna),
- **I want to:** View an automatically calculated ledger showing collected revenue, outstanding invoices, and reverse partner fees inside my Strapi command dashboard,
- **So that I can:** Monitor the platform's cash pipelines in real-time without running manual calculator checks.

**Test Rule / Acceptance Criteria:**
- **GIVEN** Anna is viewing the master administrative ledger workspace,
- **WHEN** an external partner clicks their secure link and updates a deal status to a fee-triggering milestone,
- **THEN** the backend engine must instantly pull the calculation rules from that partner's private profile, update the `referral_records` database row, recalculate the master summary balances, and display the update on the dashboard page within < 2 seconds.

### Use Story ID: US-ADM-005 (Automated Deal Stagnation Module)

- **As an:** Executive Platform Super-Administrator (Anna),
- **I want to:** See a prominent visual alert flag next to any transaction row that has sat with zero status modifications for more than 7 consecutive days,
- **So that I can:** Immediately contact that specific partner and ensure our co-buying users aren't left stranded.

**Test Rule / Acceptance Criteria:**
- **GIVEN** the background system validation loop is running its 24-hour analysis pass,
- **WHEN** a record's latest `status_history` update timestamp is calculated to be older than 7 days from the current date,
- **THEN** the system must toggle that record's alert property status to `true` and inject a red notification badge next to the entry row inside the admin workspace table view.

---

## 5. Interactive HTML Command Dashboard Engine

The `admin.html` file implements a **Live Operational Simulator**, combining app growth tracking, live user log-ins, compliance signature captures, recurring subscription revenue, and hidden partner fee calculations on a single interface page.

### How to open and run the prototype

Open `admin.html` in any standard web browser (Chrome, Safari, Edge). It runs instantly with zero external requirements.

### Layout

| Component | Description |
|-----------|-------------|
| **Header** | Vestie brand, navigation links, "Export Accounting CSV" button, Admin Session badge |
| **Module A — App Growth Metrics** | 3 KPI cards: App Store Downloads (14,350), Concurrent Active Users (420), Compliance Verified Profiles (9,410) |
| **Module B — Revenue & Ledger** | 3 KPI cards: Referral Fees Earned ($24,500), Outstanding Receivables ($12,300), Partner Payables — Reverse Owed ($1,170 with 15% kickback) |
| **Module C — Subscriptions** | 3 KPI cards: Free Tier (8,990), Premium Subscribers (420), MRR ($4,158) |
| **Module D/E — Transaction Table** | Referral pipeline with stagnation alerts (red badge for 7+ days inactive) and Click-to-Pay fee settlement |
| **Sidebar** | Simulation Controller with Simulate App Install, Simulate Active Concurrent Ping, Convert Free User to Premium, and System Operation Log Feed |

### Interactions

| Button | Effect |
|--------|--------|
| Simulate App Install & Signup | Increments downloads (+1), free users (+1), compliance (+1) |
| Simulate Active Concurrent Ping | Bumps active users by a random amount (5–29) |
| Convert Free User to Premium | Moves one free user to premium, adds $9.90 MRR |
| Click any table row (pending fee) | Settles the referral: adjusts earned/outstanding/reverse KPIs |
| Export Accounting CSV | Generates and downloads a `.csv` file of all transaction rows |

---

## 6. Strapi Schema Extensions

New fields to add to the existing `vestie_partners` collection:

| Field Name | Strapi Type | Required | Default | Notes |
|------------|-------------|----------|---------|-------|
| `plan_tier` | Enumeration | Yes | `Free` | Options: Free, Starter, Pro, Enterprise |
| `renewal_date` | Date | No | — | Next subscription renewal date |
| `subscription_status` | Enumeration | Yes | `Active` | Options: Active, Expiring, Expired, Suspended |
| `billing_type` | Enumeration | Yes | `Flat` | Options: Flat, Percent |
| `base_fee_amount` | Decimal | No | `0` | Flat fee in AUD |
| `commission_rate` | Decimal | No | `0` | Percentage as decimal (e.g. 0.015 = 1.5%) |
| `trigger_milestone` | Enumeration | Yes | — | Milestone that triggers fee calculation |

Extended `vestie_referrals` collection:

| Field Name | Strapi Type | Required | Default | Notes |
|------------|-------------|----------|---------|-------|
| `current_stage` | Integer | Yes | `1` | 1, 2, or 3 |
| `days_inactive` | Integer | Yes | `0` | Days since last partner activity |
| `risk_level` | Enumeration | No | `Low` | Low, Medium, High |
| `assigned_partner` | Relation | Yes | — | Many-to-one to `vestie_partners` |

Extended `core_user_profiles` collection:

| Field Name | Strapi Type | Required | Default | Notes |
|------------|-------------|----------|---------|-------|
| `user_uuid` | UUID | Yes | — | Unique primary key |
| `compliance_signed` | Boolean | Yes | `False` | Whether user completed compliance check |
| `compliance_timestamp` | DateTime | No | — | When compliance was signed |
| `subscription_tier` | Enumeration | Yes | `Free` | Options: Free, Premium |
| `monthly_billing_rate` | Decimal | Yes | `0.00` | AUD, per subscription tier |

---

## 7. Implementation Notes for Radial Code

**Priority:** Modules A and B are the highest priority — they demonstrate the most business value to Anna. Modules D and C are medium priority. Module E (Export) is a utility feature that should be built last but is trivial to implement.

**Simulation Strategy:** All data is hardcoded in JavaScript arrays/objects. The sidebar buttons trigger state changes to demonstrate dynamic behavior. No backend integration is required for the prototype.

**15% Kickback Rule:** Reverse Partner Fees are calculated as 15% of the referral fee value. This is stored as a calculated field in the `referral_records` table, not hardcoded in the UI.

**CSS Architecture:** All admin components use the same CSS variables (`--bg-primary`, `--bg-secondary`, etc.) defined in `index.html`. The `admin.html` file redeclares these variables at the root level for standalone use.

**CSV Export Pattern:**
```javascript
function exportCSV(tableId, filename) {
  const rows = document.querySelectorAll(`#${tableId} tbody tr`);
  const csv = [...rows].map(row =>
    [...row.querySelectorAll('td')].map(cell => `"${cell.innerText}"`).join(',')
  ).join('\n');
  const blob = new Blob([csv], { type: 'text/csv' });
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = filename;
  a.click(); URL.revokeObjectURL(url);
}
```

---

## 8. Hidden Partner Fee Configurations (Strapi Core Setup)

### 8.1 The Private Fields Strategy (Hidden from Public)

To make fee calculations effortless, administrative data fields are added directly to the end of the existing `vestie_partners` collection schema in Strapi. These fields are flagged as **"Private"** — invisible to the public and mobile application frontend — but the backend automation reads them instantly to compute invoices.

```
       [Strapi CMS: Master Partner Collection Model]
 ─────────────────────────────────────────────────────────────
  VISIBLE TO THE PUBLIC APP:
  ├── Field: full_name         [Text: e.g., "Apex Tax Specialists"]
  ├── Field: professional_role [Dropdown: "Tax Depreciation Specialist"]
  ├── Field: avatar_image      [Image Upload]
  └── Field: provider_bio      [Text Box Summary]
 ─────────────────────────────────────────────────────────────
  🔒 HIDDEN FROM PUBLIC (ADMIN/CALCULATION ONLY):
  ├── Field: billing_type      [Dropdown: "Flat Fee" OR "Percentage"]
  ├── Field: base_fee_amount   [Number/Currency: e.g., 120]
  ├── Field: commission_rate   [Number/Percentage: e.g., 0.20 for 20%]
  └── Field: trigger_milestone [Dropdown: "Engagement Confirmed", "Report Delivered", etc.]
```

**Why this matters:** When Anna onboards a new partner in Strapi, she fills out a few extra fields at the bottom of their profile page. No custom setup screens needed. No hardcoded values in the app. The Magic Link submission handler reads these fields and computes the invoice automatically.

### 8.2 Dynamic Data Fields Definition Matrix

| Strapi Field Name | Data Input Type | What Anna Selects/Inputs in Strapi | How the App Automation Handles It |
|-------------------|-----------------|-------------------------------------|--------------------------------------|
| `billing_type` | Enumeration (Dropdown) | Selects either **[Flat Fee]** or **[Percentage]** | Tells the calculation engine which mathematical path to use when a deal finishes |
| `base_fee_amount` | Decimal (Currency) | Types the exact flat fee value (e.g., **120** for Tax Specialists, **300** for Lawyers) | If billing type is "Flat Fee", the system copies this number straight to the invoice ledger |
| `commission_rate` | Decimal (Percentage) | Types the variable percentage rate (e.g., **0.015** for a 1.5% Broker commission) | If billing type is "Percentage", the system calculates: `Property Price × Commission Rate × Vestie Share (20%)` |
| `trigger_milestone` | Enumeration (Dropdown) | Selects the milestone that triggers the fee (e.g., **[Report Delivered]** or **[Pre-Approval]**) | The system holds the invoice as "Pending" until the partner hits this specific milestone via their Magic Link |

### 8.3 Fee Calculation Rules Engine

**Flat Fee Rule:**
- `Invoice Amount = base_fee_amount`
- Used for fixed-price services: Tax Depreciation ($120), Property Lawyer ($300), Building Inspector ($180)

**Percentage Commission Rule:**
- `Invoice Amount = Property Price × Loan-to-Value Ratio × commission_rate × Vestie Split (20%)`
- Used for variable-value services: Mortgage Broker (1.5% commission → ~$1,800 on a $750k property)
- Default assumptions for simulation: Property Price = $750,000, LVR = 80%

### 8.4 Trigger-to-Fee Mapping

| Milestone | Roles That Use It | When Fee Triggers |
|-----------|-------------------|-------------------|
| Report Delivered | Tax Specialist, Building Inspector, Valuer, Surveyor | Upon PDF upload and submission |
| Engagement Confirmed | Lawyer, Conveyancer, Settlement Agent | Upon contract review confirmation |
| Loan Settlement | Mortgage Broker, Financial Planner | Upon loan approval milestone |
| Pre-Approval | Mortgage Broker (alternate) | Upon lender pre-approval confirmation |
| Policy Bound | Insurance Broker | Upon policy issuance and binding |
| Strategy Implemented | Financial Planner | Upon plan implementation sign-off |

### 8.5 Interactive Prototype (`fee_config.html`)

A standalone HTML file (`fee_config.html`) validates the exact solution: storing accounting configuration metrics inside hidden fields within the **Partner Profile itself**.

**Layout:**
- **Left pane:** Phone chassis simulating partner email inbox → Magic Link webform with milestone checkbox + PDF upload → success state with fee calculation result
- **Right pane:** Strapi configuration panel with:
  - Partner profession dropdown (3 presets: Tax Specialist $120 flat, Lawyer $300 flat, Broker 1.5% percentage)
  - `billing_type`, `base_fee_amount`/`commission_rate`, `trigger_milestone` fields
  - Live ledger table showing calculated fee owed per hidden profile config
  - Terminal log

**How to present to the team:**
- **To Anna:** "Anna, look at the Strapi box on the right. Those are hidden fields where you type the $120 fee. It never shows to the public application. When a partner clicks the link on their email, the system looks at these hidden settings and automatically calculates your cash pipeline."
- **To Radial Code:** "Add private metadata fields (`billing_type`, `base_fee_amount`, `trigger_milestone`) straight to the pre-existing `vestie_partners` Strapi schema collection model, and run a calculation webhook on webform submission."
