# Magic Link Demo — Gap Analysis vs VENDOR-REQUIREMENTS.md

**Date:** 2026-06-03
**Source:** `/tmp/magic-link-demo/` (6 HTML + PRD.md, 2927 lines total)
**Reference:** `VENDOR-REQUIREMENTS.md` (378 lines)

## Demo Scope

The Magic Link demo implements a complete dark-themed prototype with:

| File | Lines | Purpose |
|------|-------|---------|
| `index.html` | 329 | Command Center — 12 partner tiles, security vault, audit log |
| `demo.html` | 765 | Magic Link form — 12 roles × 3 stages, 8 field types |
| `admin.html` | 372 | Admin Dashboard — Modules A–E, simulators, CSV export |
| `fee_config.html` | 448 | Fee Config Engine — hidden Strapi fields driving auto-calculation |
| `invoice_demo.html` | 283 | Invoice Templates — 3-template toggle (Client Fee / Service Rendered / Commission Payout) |
| `paystub_demo.html` | 305 | Unified Receivables & Paystub Engine — 3 scenarios (Legal Split / Inspection / Broker Commission) |
| `PRD.md` | 381 | Product Requirements Document — full schema reference |
| `README.md` | 44 | Quick-start guide |

## Critical Gaps (Must Fix)

### 1. Role Naming Mismatch
- **Demo uses:** `broker`, `lawyer`, `inspector`, `tax`, `agent`, `valuer`, `surveyor`, `strata`, `planner`, `insurance`, `settlement`, `conveyancer`
- **VENDOR requires:** `mortgage_broker`, `property_lawyer`, `building_inspector`, `tax_specialist`, `real_estate_agent`, `property_valuer`, `land_surveyor`, `strata_manager`, `financial_planner`, `insurance_broker`, `settlement_agent`, `licensed_conveyancer`
- **Impact:** URL params in `demo.html?role=...` would break in production unless backend normalises them

### 2. Stage Model Mismatch
- **Demo uses:** stage 1/2/3 (generic 3-phase per-role)
- **VENDOR requires:** 6-stage pipeline (Match → Financing → Agreement → Find Property → Offer → Settlement) + per-role sub-stages
- **Impact:** Integration with admin-v2 kanban impossible without mapping layer; stage numbers don't match

### 3. CSS Theme Mismatch
- **Demo:** Dark mode (`--bg-primary: #0F1117`, `--bg-secondary: #1A1D27`)
- **VENDOR / admin-v2:** Japanese minimalist (`--bg-page: #FAF8F5`, `--accent: #C4735B`)
- **Impact:** Visual inconsistency — the magic link partner form would look out of place inside Vestie's warm UI

### 4. Missing Admin Dashboard Modules
- **Demo admin.html has:** Modules A (Traffic), B (Revenue Ledger), C (Subscriptions), D (Stagnation), E (CSV Export)
- **VENDOR / admin-v2 requires 12 modules:** Overview, Users, Partners, Trust & Safety, Deals, Messages, Agreements, Listings, Documents, Billing, Notifications, Settings
- **Coverage gap:** 7 modules missing (Users, Partners, Trust & Safety, Messages, Agreements, Listings, Documents, Notifications, Settings)
- **Note:** Deals (kanban) and Billing are covered in admin-v2 but not present in the demo admin.html

### 5. Token / Magic Link Structure
- **Demo:** UUID-style tokens with 7-day expiry, single-use invalidation, HTTP 403 for expired/consumed
- **VENDOR:** Same concept but requires `vestie_ml_` prefix tokens, per-role field mappings, and re-issue from admin dashboard
- **Impact:** Token format differs; no admin re-issue UI in demo

### 6. CSV Export Format
- **Demo:** Reads table rows client-side, formats as CSV via Blob URL
- **VENDOR:** Requires column headers matching ledger schema (`billing_stream`, `asset_description`, `gross_amount`, `individual_share`, `accounting_receivable`, `ledger_clear_status`)
- **Impact:** CSV output uses different column names — would need mapping to match accounting imports

### 7. Stagnation Alerts
- **Demo:** Checks `status_history` timestamps, red badge if delta >7 days, daily background pass
- **VENDOR:** Same concept but alert bar at top of admin-v2 (not inline row badge), plus send reminder email action
- **Impact:** Alert location differs; no email follow-up in demo

## Moderate Gaps

### 8. Partner Profile Fields
- **Demo partner schema (`vestie_partners`):** `full_name`, `professional_role`, `avatar_image`, `provider_bio`, `billing_type`, `base_fee_amount`, `commission_rate`, `trigger_milestone`, `plan_tier`, `renewal_date`, `subscription_status`
- **VENDOR requires these additional fields:** `phone`, `email`, `company`, `totalFees`, `deals`, `subscription` (with `active`/`premium`/`expiring`/`suspended` states)
- **Impact:** Partner directory in admin-v2 would show incomplete data

### 9. Customer Support Page (VENDOR Section 2)
- **Demo:** No support page, no ticket system
- **VENDOR:** Requires `/support` page with form fields (category, urgency, attachment), auto-reply email, admin ticket queue
- **Impact:** No coverage — would need separate implementation

### 10. Email Templates (VENDOR Section 14)
- **Demo:** Automated receipt email (post-payment) only
- **VENDOR requires 14 templates:** Welcome (Quick/Detail), Match Confirmed, Signup Incomplete, Support Ticket (Received/Admin New), Magic Link (Partner), Broker (Listing Approved/Rejected), Finance Ready, Offer Confirmed, Settlement Complete, Agent Invitation, Stagnation Alert (Admin)
- **Impact:** 13 of 14 templates missing — only post-payment receipt covered

### 11. Broker Agent Portal (VENDOR Section 3)
- **Demo:** No broker portal — broker is just one of 12 Magic Link roles
- **VENDOR:** Requires dedicated broker portal with listing CRUD, property management, performance stats, commission dashboard
- **Impact:** No coverage — would need Strapi/Strappy implementation

## Minor Gaps

### 12. Admin Kanban for Deals
- **Demo admin.html:** Table-based deal list (no kanban)
- **VENDOR / admin-v2:** 6-column kanban with drag-to-advance, stagnation visual (red left border), click card for modal
- **Impact:** Demo admin uses a different interaction model; not a blocker but inconsistent UX

### 13. Commission Calculation
- **Demo:** `Invoice = Property Price x LVR x commission_rate x Vestie Split (20%)`
- **VENDOR:** Same formula but requires field mapping in admin-v2 settings tab, per-partner override capability
- **Impact:** Formula matches but admin override UI missing

### 14. Responsive Breakpoints
- **Demo:** Single breakpoint at 1100px (sidebar collapses)
- **VENDOR / admin-v2:** 3 breakpoints (1200, 900, 640) with hamburger + off-canvas sidebar
- **Impact:** Demo is mobile-functional but less refined

## Summary

| Severity | Count | Key Items |
|----------|-------|-----------|
| Critical | 7 | Role names, stage model, theme, 7 missing dashboard modules, token prefix, CSV columns, alert location |
| Moderate | 4 | Partner profile fields, support page, 13 email templates, broker portal |
| Minor | 3 | Kanban UX, commission overrides, responsive breakpoints |
| **Total** | **14** | |

## Recommendation

1. **Merge demo magic link form** (`demo.html`) into admin-v2 by adding it as a tab with remapped role names and Japanese minimalist theme
2. **Rebuild admin.html** functionality into admin-v2's Billing tab (already partially done — has splits, commissions, CSV export, simulators)
3. **Port demo fee_config engine** into admin-v2 Settings tab as an integration panel
4. **Port demo invoice/paystub** into a dedicated FinTech section (future phase)
5. Implement customer support page and broker portal as separate new files
