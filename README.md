# EAMI QMS — Change Control Module

A digital Change Control module for EAMI's Quality Management System (QMS). Replaces paper-based change control with a workflow-driven web application that enforces role-based permissions, captures a full audit trail, and routes every change through a defined approval lifecycle.

---

## Project Status

**Pre-development phase complete. REST API phase has not started yet.**

| # | Phase | Status |
|---|-------|--------|
| 1 | Business Requirements Document (BRD) | ✅ Approved & signed off (V1.20.04.2026) |
| 2 | Security Matrix | ✅ Finalized (V1.0) |
| 3 | State Machine + Sequence Diagrams | ✅ Finalized |
| 4 | HTML Prototypes (all states, all roles) | ✅ Complete |
| 5 | Global CSS (design system) | ✅ Complete |
| 6 | DB Design Document | ✅ Approved (V1.0, 22 April 2026) |
| 7 | Database Creation (SQL scripts + stored procedures) | ✅ Created, verified, tested |
| 8 | REST API Design & Development | ⏳ Not started |
| 9 | Frontend (Angular) | ⏳ Not started |

---

## What's in this Repo

### 1. Business Requirements Document (BRD)

The single source of truth for what the system does. Signed off on 21 April 2026. Split into 13 markdown files for readability:

- Executive Summary, scope, success criteria
- User Roles & Personas (CC Owner, Approver, Viewer, Admin)
- Workflow & States (6 states, 8 transitions, rejection flows)
- Security Matrix narrative (50 fields, field-level permissions)
- 24 user stories with acceptance criteria
- 48+ functional requirements
- Full field definitions for all 50 fields (data types, validation, dropdowns, mandatory rules)
- 62 business rules
- UI/UX guidelines, non-functional requirements, integration, limitations, assumptions, appendices

📁 `BRD_Section_1.md` → `BRD_Section_11-15.md`
📄 `BRD_V1_20_04_2026_Sign_Off.pdf` — signed approval document
📄 `BRD_HANDOFF_DOCUMENT.md` — gap resolutions and business decisions

### 2. Security Matrix

Defines who can do what, where, and when. **50 fields × 6 states × 4 roles**, colour-coded (green = editable, red = read-only). Drives field-level permissions throughout the application.

📄 `Security_Matrix_V1_0.xlsx`

### 3. Diagrams

- **State Machine Diagram** — 6 states (Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, Cancelled), all transitions including rejection loops, and the editable fields per state per role.
- **Sequence Diagram** — Full CC Owner ↔ System ↔ Approver interaction across all four workflow phases, including both rejection scenarios.

📄 `Flow_Chart.png` + `Flow_Chart_Code.txt` (Mermaid)
📄 `Sequence_Diagram.png` + `Sequence_Diagram_Code.txt` (Mermaid)

### 4. HTML Prototypes

Comprehensive UI prototypes covering **every workflow state from every role's perspective**. Built with semantic HTML and the global CSS design system. These are not throwaway mockups — the Angular frontend will reuse this markup and styling directly.

**Authentication & onboarding:**
- `login.html`, `forgot-password.html`, `reset-password.html`, `email-reset-password.html`

**Dashboards (per role):**
- `dashboard-cc-owner.html`, `dashboard-approver.html`, `dashboard-empty.html`

**List views:**
- `all-change-controls.html`, `my-change-controls.html`, `my-change-controls-empty.html`
- `approvals.html`, `approvals-empty.html`

**Change Control Form (every state × every relevant role):**
- `cc-form-initated-state.html` — CC Owner view (Initiated)
- `cc-form-initated-state-approver-view.html` — Approver view (Initiated)
- `cc-form-pending-implementation-approval-user-view.html` — CC Owner view
- `cc-form-pending-implementation-approval-approver-view.html` — Approver view
- `cc-form-in-implementation-implementer-view.html` — CC Owner view
- `cc-form-in-implementation-approver-view.html` — Approver view
- `cc-form-pending-final-approval-implementer-view.html` — CC Owner view
- `cc-form-pending-final-approval-approver-view.html` — Approver view
- `cc-form-closed.html`, `cc-form-cancelled.html` — terminal states

**Settings:**
- `settings-admin.html`, `settings-profile.html`, `settings-profile-enduser.html`

### 5. Global CSS

Flat enterprise design system (~34KB). Variables, typography, spacing, components, and form patterns. Single source for all styling — every HTML prototype imports it.

📄 `global.css`

### 6. Database Design Document

Comprehensive DB design split across 10 markdown sections covering ERD, table specifications, columns, indexes, constraints, lookup data, the 3 stored procedures, security considerations, and the BRD-field-to-DB-column mapping.

📁 `DB_Design_Document_Section_1.md` → `DB_Design_Document_Section_10.md`
📄 `DB_DESIGN_HANDOFF_DOCUMENT.md` — architecture decisions

### 7. Database Scripts

The actual SQL Server database — built, verified, and tested. Includes all tables, constraints, foreign keys, indexes, lookup/seed data, and 3 stored procedures (`usp_InsertAuditLog`, `usp_CheckActiveRecordsForUser`, `usp_GenerateCCID`).

📄 `EAMI_QMS_ChangeControl_Full_Script.sql`
📄 `DB_Scripts_Creation_Guide.md`

---

## Architecture Approach

**DB-first, hybrid business logic (Option C):**

- **Database** — structural safety only: constraints, foreign keys, defaults, auto-generated fields, audit log INSERT (via stored procedure).
- **C# service layer** — all business logic: state transitions, field validation, mandatory field checks, role-based permission checks, edge case rules, email notification triggers, file upload handling.

This keeps the database clean and the business rules unit-testable in C#.

---

## Planned Tech Stack

| Layer | Technology |
|-------|-----------|
| Database | SQL Server 2022 Express |
| ORM | Entity Framework Core (DB-first, scaffolded) |
| Backend | .NET 8 / ASP.NET Core |
| Authentication | JWT (BCrypt for password hashing) |
| Validation | FluentValidation |
| Mapping | AutoMapper |
| Logging | Serilog (structured logs, daily rolling files, CorrelationId) |
| API Docs | Swagger / Swashbuckle |
| Frontend | Angular (reusing existing HTML/CSS) |
| Hosting | IIS |

---

## What's Next

A quick peek at the upcoming phases — none of these have started yet:

1. **REST API Design Document** — endpoint catalog, DTO definitions, validation rules, error response contracts.
2. **REST API Development** — .NET 8 backend with the standard 3-layer pattern (Controllers → Services → Repositories), JWT auth, Serilog logging, FluentValidation, AutoMapper, Swagger.
3. **API Testing** — Swagger + Postman test collections covering happy paths, validation failures, permission checks, and full workflow scenarios.
4. **Frontend Development** — Angular application built on the existing HTML prototypes and `global.css`, wired to the REST API.
5. **End-to-end testing & deployment** — IIS hosting, environment configuration, UAT.

---

## Repo Map

```
.
├── BRD_Section_*.md                  # Business Requirements Document (13 files)
├── BRD_V1_20_04_2026_Sign_Off.pdf    # Signed BRD approval
├── BRD_HANDOFF_DOCUMENT.md           # Gap resolutions & business decisions
│
├── Security_Matrix_V1_0.xlsx         # 50 fields × 6 states × 4 roles
│
├── Flow_Chart.png / .txt             # State machine diagram (Mermaid)
├── Sequence_Diagram.png / .txt       # Sequence diagram (Mermaid)
│
├── *.html                            # 25 HTML prototypes (auth, dashboards, lists, forms, settings)
├── global.css                        # Design system / global styles
│
├── DB_Design_Document_Section_*.md   # DB design (10 sections)
├── DB_DESIGN_HANDOFF_DOCUMENT.md     # DB architecture decisions
├── DB_Scripts_Creation_Guide.md      # How to run the SQL script
└── EAMI_QMS_ChangeControl_Full_Script.sql  # Full DB creation script
```

All comprehensive documentation lives in this repo — every business rule, field definition, state transition, permission rule, and DB column mapping is documented.

---

## Document Versions

| Document | Version | Date |
|----------|---------|------|
| BRD | 1.20.04.2026 | 21 April 2026 |
| Security Matrix | 1.0 | — |
| DB Design Document | 1.0 | 22 April 2026 |

---

**EAMI QMS — Change Control Module** · Phase 1
