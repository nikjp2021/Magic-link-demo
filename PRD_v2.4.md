# PRODUCT REQUIREMENTS DOCUMENT (PRD) ADDENDUM v2.4

## Admin Operations Dashboard & Hidden Partner Fee Configurations

| Document | Details |
|----------|---------|
| **Project** | Vestie Magic Link Architecture |
| **Version** | 2.4 (Addendum) |
| **Status** | Draft for Client Review |
| **Target Audience** | Anna (Operations Admin) & Radial Code (Dev Team) |

---

## Sign-Off

| Role | Name | Date | Signature |
|------|------|------|-----------|
| Product Owner | Anna | TBD | TBD |
| Lead Developer | Radial Code | TBD | TBD |
| UX Consultant | Designer | TBD | TBD |

---

## Table of Contents

1. [Admin Operations Dashboard (Modules A–E)](#1-admin-operations-dashboard-modules-a-e)
   - [Module A: Live Traffic & Pipeline Monitor](#module-a-live-traffic--pipeline-monitor)
   - [Module B: Fee & Ledger Reconciliation](#module-b-fee--ledger-reconciliation)
   - [Module C: Subscription Status Grid](#module-c-subscription-status-grid)
   - [Module D: Stagnation & Risk Alerts](#module-d-stagnation--risk-alerts)
   - [Module E: Partner Export & CSV Extraction](#module-e-partner-export--csv-extraction)
2. [Component Consolidation Matrix](#2-component-consolidation-matrix)
3. [Use Case Specification Grid](#3-use-case-specification-grid)
4. [User Stories](#4-user-stories)
5. [Interactive Prototype Notes (admin.html)](#5-interactive-prototype-notes-adminhtml)
6. [Strapi Schema Extensions](#6-strapi-schema-extensions)
7. [Implementation Notes for Radial Code](#7-implementation-notes-for-radial-code)
8. [Hidden Partner Fee Configurations (Strapi Core Setup)](#8-hidden-partner-fee-configurations-strapi-core-setup)
   - [8.1 The Private Fields Strategy](#81-the-private-fields-strategy-hidden-from-public)
   - [8.2 Dynamic Data Fields Definition Matrix](#82-dynamic-data-fields-definition-matrix)
   - [8.3 Fee Calculation Rules Engine](#83-fee-calculation-rules-engine)
   - [8.4 Trigger-to-Fee Mapping](#84-trigger-to-fee-mapping)
   - [8.5 Interactive Prototype (fee_config.html)](#85-interactive-prototype-fee_confightml)

---

## 1. Admin Operations Dashboard (Modules A–E)

### Overview

A standalone admin dashboard (`admin.html`) that replaces the "Admin Operations" section originally described in PRD v2.2. This dashboard is a **separate static page** (not merged into the Command Center) designed for Anna to monitor, reconcile, and manage the entire partner ecosystem at a glance.

**Core Philosophy:** Admin-only session — no partner tiles, no Magic Link forms, no mobile simulation. Pure operational data views with simulation controls on the right sidebar.

---

### Module A: Live Traffic & Pipeline Monitor

**Purpose:** Provide Anna with a real-time (simulated) overview of how many partners are active, what engagement stages they're in, and the health of the overall pipeline.

**UI Components:**
- **KPI Bar** — 4 stat cards at the top of the dashboard:
  - **Active Referrals** — Total open referral files currently in the system (simulated: 847)
  - **Pending Verifications** — Partners who received a Magic Link but haven't submitted yet (simulated: 34)
  - **Verified Today** — Successful submissions in the last 24h (simulated: 12)
  - **Escalated** — Files flagged for manual admin review due to expiry or mismatch (simulated: 3)
- **Traffic Timeline** — A segmented bar chart showing referral volume by partner role for the current week. Segments: Broker (32), Lawyer (28), Inspector (18), Tax (12), Agent (22), Valuer (8), Surveyor (5), Strata (6), FinPlanner (4), Insurance (3), Settlement (7), Conveyancer (2).
  - Each segment is a `<div>` with a color matching the partner's tag color, proportional width by count, with hover tooltip showing role + count.
- **Pipeline Funnel** — A 3-stage visual funnel showing the number of partners at each milestone stage across all roles:
  - Stage 1 (Initiated): 412
  - Stage 2 (In Progress): 298
  - Stage 3 (Completed): 137

**States:**
- **Default:** Shows the simulated data above with no live backend connection.
- **Empty:** If no referrals exist, display an empty-state illustration with text "No active referrals yet. Partner activity will appear here once Magic Links are issued."

---

### Module B: Fee & Ledger Reconciliation

**Purpose:** Show Anna an aggregated financial view of all fees incurred, paid, and pending across the partner ecosystem.

**UI Components:**
- **Financial KPI Bar** — 3 stat cards:
  - **Total Invoiced (MTD):** $12,450 AUD
  - **Total Collected:** $9,830 AUD
  - **Outstanding:** $2,620 AUD
- **Transaction Ledger Table** — Full-width sortable table with columns:
  - Referral ID, Partner Name, Role, Fee Type, Amount, Status, Date
  - Simulated rows (min 5) showing a mix of Flat Fee and Percentage billing types
  - Status badges: `Paid` (green), `Pending` (yellow), `Overdue` (red)
- **Fee Distribution Chart** — A visual breakdown (simple stacked bar or horizontal bars) of MTD fees by partner role, so Anna can see which roles generate the most revenue.

**States:**
- **Default:** Populated with simulated ledger data.
- **Empty:** "No transactions recorded this period. Fees will appear once partners complete milestones."
- **Loading:** A brief skeleton loader while "fetching" data.

---

### Module C: Subscription Status Grid

**Purpose:** Give Anna a bird's-eye view of every partner's subscription tier, renewal date, and account status in one scrollable grid.

**UI Components:**
- **Grid Table** with columns:
  - Partner Name, Role, Plan Tier (Free / Starter / Pro / Enterprise), Renewal Date, Status (Active / Expiring / Expired / Suspended)
  - Simulated 12 rows (one per partner role)
  - Row color coding: green tint for Active, yellow for Expiring, red for Expired/Suspended
- **Plan Distribution Summary** — A small stat row above the grid:
  - Free: 3 | Starter: 4 | Pro: 4 | Enterprise: 1

**States:**
- **Default:** Grid populated with 12 simulated partner records.
- **Empty:** "No partners subscribed yet. Subscription data will populate after partner onboarding."

---

### Module D: Stagnation & Risk Alerts

**Purpose:** Surface partners or referral files that have stalled — no Magic Link activity, expired tokens, or incomplete stages.

**UI Components:**
- **Risk Summary Cards** — 3 cards:
  - **Stalled Referrals:** Referral files with no partner activity for 7+ days (simulated: 18)
  - **Expired Tokens:** Magic Link tokens that reached their 7-day expiry without submission (simulated: 7)
  - **Incomplete Stage 1:** Partners who opened a Magic Link but never completed the first stage (simulated: 11)
- **Stagnation Table** — Detailed list with columns:
  - Referral ID, Partner, Role, Current Stage, Days Inactive, Risk Level (Low / Medium / High)
  - Risk levels: High = 14+ days inactive (red), Medium = 7–13 days (yellow), Low = <7 days (green)
  - Simulated 5 rows covering various risk levels

**States:**
- **Default:** Shows stagnation data.
- **Empty:** "No stagnation alerts. All partner workflows are progressing normally."
- **Alert State:** An edge case where a single partner accounts for multiple stalled referrals — show "3 stalled referrals from [Partner Name] — recommend direct outreach."

---

### Module E: Partner Export & CSV Extraction

**Purpose:** Allow Anna to export any of the above data views to a CSV file that can be opened in Excel or imported into accounting software (Xero, MYOB).

**UI Components:**
- **Export Buttons** — One per module (A–D) labeled "Export CSV" placed at the top-right of each panel header
- **CSV Generation** — JavaScript function that reads the current table data and triggers a download via `Blob` + `URL.createObjectURL`
- **Export Log** — A small terminal/log area in the sidebar that records each export event with timestamp

**States:**
- **Default:** Export buttons active and clickable.
- **Empty Module:** If the table is empty, the export button is disabled with a tooltip "No data to export."
- **Download Success:** Toast notification "CSV exported: [module]_[timestamp].csv"

---

## 2. Component Consolidation Matrix

This table maps each admin module to its type, the data source it depends on, and its corresponding Strapi entity.

| # | Module Name | Component Type | Data Source | Strapi Entity |
|---|-------------|---------------|-------------|---------------|
| A | Live Traffic & Pipeline Monitor | KPI Cards + Charts | Aggregated referral counts | `vestie_referrals` with partner join |
| B | Fee & Ledger Reconciliation | Financial Table + Charts | Fee calculation engine output | `vestie_invoices` computed from partner billing fields |
| C | Subscription Status Grid | Table with color-coded rows | Partner subscription records | `vestie_partners.plan_tier` |
| D | Stagnation & Risk Alerts | Cards + Alert Table | Magic Link activity timestamps | `vestie_magic_links` + `vestie_audit_log` |
| E | Partner Export & CSV Extraction | Button + Blob download | All of the above | Dynamic CSV serialization |

---

## 3. Use Case Specification Grid

| ID | Use Case | Primary Actor | Trigger | Precondition | Postcondition |
|----|----------|---------------|---------|--------------|---------------|
| UC-ADM-01 | View pipeline traffic | Anna | Opens admin dashboard | Valid admin session | Dashboard renders with simulated stats |
| UC-ADM-02 | Reconcile fee ledger | Anna | Clicks Module B panel | Fees exist for current period | Ledger displays with correct totals |
| UC-ADM-03 | Check subscription status | Anna | Clicks Module C panel | Partners exist in system | Status grid loads with tier info |
| UC-ADM-04 | Identify stalled referrals | Anna | Clicks Module D panel | Stagnation data exists | Risk alerts display sorted by severity |
| UC-ADM-05 | Export ledger to CSV | Anna | Clicks "Export CSV" on Module B | Ledger has data rows | CSV file downloads; event logged |
| UC-ADM-06 | Export stagnation report | Anna | Clicks "Export CSV" on Module D | Stagnation table has rows | CSV file downloads; event logged |
| UC-ADM-07 | Simulate partner fee config | Anna / Demo | Changes dropdown in fee_config.html | fee_config.html loaded | Ledger, phone view, terminal update in real time |

---

## 4. User Stories

### US-ADM-004: Accounting Fee Reconciliation View

> **As an** operations administrator (Anna),
> **I want** to see a live reconciliation dashboard that displays all fees invoiced, collected, and outstanding,
> **so that** I can track revenue flow and identify unpaid balances without logging into accounting software.

**Acceptance Criteria:**
- Dashboard shows MTD totals for invoiced, collected, and outstanding fees
- Transaction ledger displays individual fee items with partner name, role, fee type, amount, and status
- Fee distribution chart breaks down revenue by partner role
- Each table/section has an Export CSV button
- All data is simulated (no live backend) but structured as real API responses would be

### US-ADM-005: Partner Stagnation Monitoring

> **As an** operations administrator (Anna),
> **I want** to view a stagnation report showing which partners have stalled workflows or expired Magic Links,
> **so that** I can proactively reach out and keep the pipeline moving.

**Acceptance Criteria:**
- Dashboard shows count of stalled referrals, expired tokens, and incomplete Stage 1 items
- Detailed stagnation table lists referral ID, partner, role, current stage, days inactive, and risk level
- Risk levels are color-coded (red/yellow/green)
- Export CSV button available for stagnation data
- Empty state shown when no stagnation exists

---

## 5. Interactive Prototype Notes (admin.html)

A standalone file `admin.html` implements all five modules (A–E) in a single-page dashboard layout:

**Layout:**
- Full-width dark-themed dashboard (consistent with `index.html` and `demo.html`)
- **Left column (main):** 5 stacked panel cards for Modules A–E
- **Right sidebar (sticky):** Simulation controller with:
  - "Refresh Data" button (re-shuffles simulated values for demo effect)
  - "Reset to Defaults" button
  - Export log showing last 5 CSV exports
- **Global header:** Vestie Admin badge, "Operations Dashboard" title, session indicator

**Technical Notes:**
- All data is client-side simulated — no fetch calls to any backend
- CSV export uses `Blob` + `URL.createObjectURL` with `text/csv` MIME type
- Charts are pure CSS/HTML (no Chart.js or external libraries) — segmented bars and colored divs
- Responsive breakpoints match the existing codebase at 1024px, 768px, 480px
- Zero external dependencies — runs entirely offline in any browser

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

---

## 7. Implementation Notes for Radial Code

**Priority:** Modules A and B are the highest priority — they demonstrate the most business value to Anna. Modules D and C are medium priority. Module E (Export) is a utility feature that should be built last but is trivial to implement.

**Simulation Strategy:** All data is hardcoded in JavaScript arrays/objects. The "Refresh Data" button randomizes values within realistic ranges to demonstrate dynamic behavior. No backend integration is required for the prototype.

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
