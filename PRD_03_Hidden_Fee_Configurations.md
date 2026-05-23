# PRODUCT REQUIREMENTS DOCUMENT (PRD)

**Module:** Hidden Partner Fee Configurations
**Project:** Vestie Mobile Application & Operational Command Center
**Version:** 2.4 (Section 8)
**Author:** Nikhil Tiwari (Product Manager)
**Date:** 23/05/2026

---

## Sign-Off

| Role | Name | Signature / Approval Status | Date |
|------|------|----------------------------|------|
| **Product Owner (Client)** | [Client Name] | *Awaiting Review* | **/**/2026 |
| **Product Manager** | Nikhil Tiwari | Approved | 23/05/2026 |
| **Development Lead** | Radial Code Team | *Awaiting Engineering Review* | **/**/2026 |

---

## 1. The Private Fields Strategy (Hidden from Public)

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

---

## 2. Dynamic Data Fields Definition Matrix

| Strapi Field Name | Data Input Type | What Anna Selects/Inputs in Strapi | How the App Automation Handles It |
|-------------------|-----------------|-------------------------------------|--------------------------------------|
| `billing_type` | Enumeration (Dropdown) | Selects either **[Flat Fee]** or **[Percentage]** | Tells the calculation engine which mathematical path to use when a deal finishes |
| `base_fee_amount` | Decimal (Currency) | Types the exact flat fee value (e.g., **120** for Tax Specialists, **300** for Lawyers) | If billing type is "Flat Fee", the system copies this number straight to the invoice ledger |
| `commission_rate` | Decimal (Percentage) | Types the variable percentage rate (e.g., **0.015** for a 1.5% Broker commission) | If billing type is "Percentage", the system calculates: `Property Price × Commission Rate × Vestie Share (20%)` |
| `trigger_milestone` | Enumeration (Dropdown) | Selects the milestone that triggers the fee (e.g., **[Report Delivered]** or **[Pre-Approval]**) | The system holds the invoice as "Pending" until the partner hits this specific milestone via their Magic Link |

---

## 3. Fee Calculation Rules Engine

**Flat Fee Rule:**
- `Invoice Amount = base_fee_amount`
- Used for fixed-price services: Tax Depreciation ($120), Property Lawyer ($300), Building Inspector ($180)

**Percentage Commission Rule:**
- `Invoice Amount = Property Price × Loan-to-Value Ratio × commission_rate × Vestie Split (20%)`
- Used for variable-value services: Mortgage Broker (1.5% commission → ~$1,800 on a $750k property)
- Default assumptions for simulation: Property Price = $750,000, LVR = 80%

---

## 4. Trigger-to-Fee Mapping

| Milestone | Roles That Use It | When Fee Triggers |
|-----------|-------------------|-------------------|
| Report Delivered | Tax Specialist, Building Inspector, Valuer, Surveyor | Upon PDF upload and submission |
| Engagement Confirmed | Lawyer, Conveyancer, Settlement Agent | Upon contract review confirmation |
| Loan Settlement | Mortgage Broker, Financial Planner | Upon loan approval milestone |
| Pre-Approval | Mortgage Broker (alternate) | Upon lender pre-approval confirmation |
| Policy Bound | Insurance Broker | Upon policy issuance and binding |
| Strategy Implemented | Financial Planner | Upon plan implementation sign-off |

---

## 5. Interactive Prototype (`fee_config.html`)

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

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 23/05/2026 | 2.4.0 | Initial spec for Hidden Partner Fee Configurations — private fields strategy, calculation rules, trigger-to-fee mapping, interactive prototype. |
