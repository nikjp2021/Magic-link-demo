# PRODUCT REQUIREMENTS DOCUMENT (PRD)

**Module:** Invoice Generation & Payment Scope Validation
**Project:** Vestie Mobile Application & Operational Command Center
**Version:** 2.4 (Section 9)
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

## Table of Contents

1. [Core System Rules for MVP Billing](#1-core-system-rules-for-mvp-billing)
2. [Component Consolidation & Invoicing Matrix](#2-component-consolidation--invoicing-matrix)
3. [Functional Requirements & Database Fields](#3-functional-requirements--database-fields)
4. [MVP User Stories & Acceptance Criteria](#4-mvp-user-stories--acceptance-criteria)

---

## 1. Core System Rules for MVP Billing

1. **No Automated B2B Payment Gateways:** For the 8-week launch scope, the system will *not* dynamically pull money from external partner bank accounts. The platform calculates and logs invoice data records; collection management is handled externally by the System Administrator (Anna).
2. **Fixed-Ratio User Splits:** All user-facing transactional invoices (Co-ownership document setup fees and fixed-fee specialist services) are bound to a default 50/50 calculation split parameter across the linked User Pair, unless manually overridden by Admin parameters.

```
       [The Two Invoicing Streams in Vestie]
 ─────────────────────────────────────────────────────────────
  STREAM 1: User-Facing Shared Cost Splits (In-App Payment)
  ├── Setup / Agreement Fees ──► Split 50/50 ──► Paid via Mobile App
  └── Fixed Partner Fees      ──► Split 50/50 ──► Paid via Mobile App
 ─────────────────────────────────────────────────────────────
  STREAM 2: Partner-Facing Commission Ledger (External Billing)
  └── Referral Commissions    ──► Auto-Calculated ──► Invoiced manually by Anna
```

---

## 2. Component Consolidation & Invoicing Matrix

| Invoice Type | How the Amount is Calculated | Back-end System Behavior | Admin UI/UX Benefit | Radial Code Build Protection |
|--------------|------------------------------|--------------------------|---------------------|------------------------------|
| **User-Facing Shared Cost Split** | `Total Fixed Cost ÷ 2 = Individual Share Owed` | Generates two matching child invoice records linked to one parent transaction ID. Monitors individual payment states. | Anna can instantly see if User A has paid their half while User B is still pending. | Uses a standard single-payment gateway integration (e.g., standard Stripe API) without complex payout logic. |
| **Partner-Facing Commission Record** | **Flat Fees:** Taken from private partner profiles. **Variable Fees:** `Property Price × Broker Rate × 20%`. | Calculates the absolute dollar fee immediately when a partner clicks their Magic Link and triggers the target milestone. | The dashboard acts as an automated cash register, displaying a clear list of who to bill at the end of the month. | Eliminates the massive security and compliance overhead of building automated vendor billing software. |

---

## 3. Functional Requirements & Database Fields

```
       [Strapi Backend Core: Invoice Ledger Collection Schema]
 ─────────────────────────────────────────────────────────────
  Collection Identifier: invoice_records
 ─────────────────────────────────────────────────────────────
  ├── Field: invoice_uuid       [DataType: UUID, Constraints: Unique PK]
  ├── Field: parent_referral_id [Linked to referral_records Collection]
  ├── Field: invoice_type       [DataType: Enumeration, Options: User_Split, Partner_Commission]
  ├── Field: gross_amount       [DataType: Decimal, Currency: AUD]
  ├── Field: individual_share   [DataType: Decimal, Currency: AUD, Default: 0.00]
  └── Field: invoice_status     [DataType: Enumeration, Options: Unpaid, Paid]
```

| S No | Module | Use Case / Screen | User Role | Description (Functional Specifications) | Visuals |
|-------|--------|-------------------|-----------|------------------------------------------|---------|
| **5.1** | FinTech | User In-App Payment Split | Mobile User | When a fixed service or agreement fee is triggered on the Property Path, the app displays a clear itemized payment request card. It shows the user their exact 50% split amount, allowing them to tap and execute the payment directly using Apple Pay, Google Pay, or Credit Card. | `[Image 28 - Payment Component Overlay]` |
| **5.2** | FinTech | Admin Commission Invoice Log | System Admin | A dedicated tracking ledger inside your Strapi dashboard website. It aggregates all system-calculated partner commissions. Displays the unique invoice ID, partner name, amount calculated, and highlights a toggle button to mark the transaction as settled once bank funds clear. | `[Admin Website - Financial Invoice Sheet]` |

---

## 4. MVP User Stories & Acceptance Criteria

### Use Story ID: US-FIN-006 (User Shared Cost Split Module)

- **As a:** Registered Co-Buyer User on the Property Path,
- **I want to:** View my itemized 50% split invoice and pay it directly inside my mobile app interface,
- **So that I can:** Settle my share of the legal setup costs easily without manually sending money bank transfers to my match partner.

**Test Rule / Acceptance Criteria:**
- **GIVEN** a shared cost milestone (e.g., *Co-ownership Agreement Setup*) is unlocked on the Property Path,
- **WHEN** User A opens their mobile screen dashboard view,
- **THEN** the app must display an individual 50% payment action button (`Total Cost $600 → Your Share $300`), isolate their payment processing state from User B, and update their specific database row status to `Paid` immediately upon a successful transaction response hook.

### Use Story ID: US-FIN-007 (Partner Commission Recording Module)

- **As an:** Executive Platform Super-Administrator (Anna),
- **I want to:** Have the system lock a permanent, un-editable commission invoice line item the moment a partner hits a trigger milestone via their Magic Link,
- **So that I can:** Export this clear data to my external accounting app (Xero) and protect Vestie from recalculation errors if property criteria change later.

**Test Rule / Acceptance Criteria:**
- **GIVEN** an external partner completes a fee-triggering event checklist via their Magic Link web page,
- **WHEN** the form submission data payload strikes the server API layer,
- **THEN** the backend calculation engine must instantly lock a static monetary value row inside the `invoice_records` collection matching the stored partner configurations, apply a status state of `Unpaid`, and present it clearly on Anna's admin ledger dashboard spreadsheet view.

---

## Revision History

| Date | Version | Changes |
|------|---------|---------|
| 23/05/2026 | 2.4.1 | Initial spec for Invoice Generation & Payment Scope Validation — two-stream invoicing, `invoice_records` schema, US-FIN-006, US-FIN-007. |
