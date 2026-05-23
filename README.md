# Magic Link Partner Verification Demo

Interactive HTML prototype demonstrating a **single-template, multi-role Magic Link system** for partner-driven verification workflows.

## What It Does

12 partner roles across 4 property transaction phases each have a 3-stage verification form — all driven by **one reusable template** that adapts based on URL parameters (`?role=xxx&stage=n`). No duplicate screens, no extra builds.

## Demo Structure

| File | Purpose |
|------|---------|
| `index.html` | Command Center — dark UI dashboard with 12 partner tiles, security vault panel, and real-time audit log |
| `demo.html` | Universal mobile form — 8 field types (yes/no, text, textarea, number, date, select, radio, checkbox), per-stage field sets, inline validation, and Strapi JSON payload output |
| `admin.html` | Admin Operations Dashboard — pipeline traffic, fee ledger, subscription grid, stagnation alerts, and CSV export (Modules A–E) |
| `fee_config.html` | Partner Fee Configuration Engine — interactive split-pane prototype demonstrating hidden Strapi profile fields driving automatic fee calculation |
| `invoice_demo.html` | Invoice Template Demo — Template A (User Split Card) and Template B (Commission Statement) with toggle, payment, and verification simulation |
| `paystub_demo.html` | Post-Payment Engine — checkout, on-screen paystub, automated email dispatch, and Financial Hub (3-step cycle) |
| `PRD.md` | Consolidated product requirements document — Magic Link system, admin dashboard, fee config, invoice templates, post-payment flow, Strapi schema, user stories |

## How to Run

1. Open `index.html` in any browser
2. Click any partner tile to launch its Magic Link demo
3. Use the Demo Controller to switch roles and cycle through stages
4. Fill in fields, upload a document, and submit to see the data payload

## 12 Partner Roles

| Phase | Roles |
|-------|-------|
| **Acquisition** | Real Estate Agent |
| **Finance** | Mortgage Broker, Financial Planner, Insurance Broker |
| **Due Diligence** | Building Inspector, Property Valuer, Land Surveyor, Tax Specialist, Strata Manager |
| **Legal & Settlement** | Property Lawyer, Licensed Conveyancer, Settlement Agent |

## Key Features

- 12 roles × 3 stages = 36 unique form views from one template
- 8 dynamic field types with inline validation
- Per-stage custom field layouts
- 7-day Magic Link validity simulation
- Real-time terminal logging (mimics Strapi audit trail)
- Zero external dependencies — plain HTML/CSS/JS, runs entirely offline
