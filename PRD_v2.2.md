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

**Solution:** A Magic Link System. When a user pair hits a milestone, the system fires an automated email containing a secure one-click link. The partner taps it, completes a 2-step form in their mobile browser, and the Strapi database updates instantly. No login, no friction.

### 1.2 Lifecycle Flow

```
[1] Admin assigns User Pair to Partner via Command Center
         |
[2] System fires automated email with unique magic link
         |
[3] Partner taps link → single webform loads (role-detected via URL)
         |
[4] Partner selects milestone status + uploads PDF proof
         |
[5] Strapi records update → Property Path advances in mobile app
```

### 1.3 Cost-Saving Principle

One template. Twelve roles. Zero duplicate screens.

The smart webform reads `?role=xxx` from the URL and dynamically renders role-specific labels, options, and file requirements. This eliminates 11 separate page builds.

---

## 2. The 12-Role Dynamic Template Matrix

| # | Partner | URL Param | Form Title | Stage 1 | Stage 2 | Stage 3 | Upload Target |
|---|---------|-----------|------------|---------|---------|---------|---------------|
| 1 | Mortgage Broker | `broker` | Verify Loan Status | Pre-Approval Submitted | Loan Approved | Funded | Pre-Approval Letter |
| 2 | Property Lawyer | `lawyer` | Verify Contract Review | Contract Under Review | Exchanged & Signed | Settlement Completed | Signed Contract Copy |
| 3 | Building Inspector | `inspector` | Verify Asset Condition | Inspection Scheduled | Report Generated | Issues Resolved | Inspection Report |
| 4 | Tax Specialist | `tax` | Verify Depreciation Setup | Depreciation Setup | Ledger Finalized | Submitted to ATO | Depreciation Ledger |
| 5 | Real Estate Agent | `agent` | Verify Property Listing | Listing Active | Offer Accepted | Under Contract | Signed Agency Agreement |
| 6 | Property Valuer | `valuer` | Verify Valuation Report | Valuation Ordered | Site Visit Done | Report Complete | Valuation Certificate |
| 7 | Land Surveyor | `surveyor` | Verify Survey Completion | Survey Ordered | Boundary Marked | Plan Lodged | Survey Plan |
| 8 | Strata Manager | `strata` | Verify Strata Clearance | Records Requested | Certificate Issued | Clearance Given | Strata Certificate |
| 9 | Financial Planner | `finplanner` | Verify Financial Strategy | Strategy Drafted | Approved by Client | Implemented | Signed Strategy Doc |
| 10 | Insurance Broker | `insurance` | Verify Insurance Cover | Quote Requested | Policy Issued | Policy Bound | Policy Document |
| 11 | Settlement Agent | `settlement` | Verify Settlement Progress | File Opened | Documents Prepared | Settled | Settlement Statement |
| 12 | Licensed Conveyancer | `conveyancer` | Verify Conveyancing Progress | Search Complete | Section 32 Reviewed | Ready to Exchange | Section 32 Statement |

---

## 3. Interactive Prototype

### 3.1 File Structure

```
D:\Viestie\
  index.html      — Command Center dashboard (role grid, vault sim, audit log)
  demo.html       — Universal mobile form (all 12 roles via URL params)
  PRD_v2.2.md     — This document
```

### 3.2 How to Run

1. Open `index.html` in any browser — launches the Command Center
2. Click any partner tile — opens `demo.html?role=<id>&stage=1` in a new tab
3. Use the Demo Controller panel to switch roles and cycle through stages
4. The phone viewport updates in real time reflecting the selected role + stage

### 3.3 URL Scheme

```
demo.html?role=broker&stage=1   → Mortgage Broker, Stage 1
demo.html?role=lawyer&stage=3   → Property Lawyer, Stage 3
demo.html?role=inspector&stage=2 → Building Inspector, Stage 2
```

### 3.4 Key Demo Features

- **12 role tiles** in Command Center with colour-coded tags
- **Dynamic form** — title, pills, upload label, file hint all change per role
- **3-stage stepper** per role with dot indicators
- **File upload** simulation with document naming convention
- **Submit flow** with success overlay and Strapi audit log entry
- **Secure token countdown** with expiration and HTTP 403 simulation
- **Real-time terminal** logging all events as Strapi would

---

## 4. Technical Notes for Radial Code

### 4.1 Component Architecture

```
Magic Link Template
  ├── Header (brand + role tag)
  ├── Form Card
  │   ├── Title (dynamic via role data)
  │   ├── Pill Selector (milestone status, 3 options)
  │   ├── File Upload Zone (single PDF, 10MB max)
  │   └── Submit Button
  └── Footer (security badge)
```

### 4.2 Strapi Data Model Mapping

| Strapi Collection | Field | Source |
|-------------------|-------|--------|
| `user_pairs` | `id` | System |
| `partner_verifications` | `role_type` | URL param |
| `partner_verifications` | `milestone_stage` | Pill selection |
| `partner_verifications` | `document_url` | Upload result |
| `partner_verifications` | `status` | Submitted/Complete |
| `property_paths` | `current_stage` | Updated on verification |

### 4.3 Security Implementation

- Pre-signed URLs with 5-minute TTL
- Single-use token invalidated after first access
- HTTP 403 returned on expired token
- All uploads validated to PDF format, max 10MB
- Audit trail logged to Strapi `verification_logs`

---

## 5. Revision History

| Date | Version | Changes |
|------|---------|---------|
| 22/05/2026 | 2.2.0 | Full 12-role matrix, interactive HTML prototype, Command Center, Magic Link specification |
