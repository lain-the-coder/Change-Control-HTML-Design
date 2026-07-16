# EA QMS — Change Control Module

A digital Change Control module for EA's Quality Management System (QMS). Replaces paper-based change control with a workflow-driven web application that enforces role-based permissions, captures a full audit trail, requires electronic signatures on every decision, and routes every change through a defined approval lifecycle.

## Project Status

**Pre-development phase complete.** Backend implementation has not started yet.

| # | Phase | Status |
|---|-------|--------|
| 1 | Business Requirements Document (BRD) | ✅ Complete (V1.1 — electronic signatures) |
| 2 | Security Matrix | ✅ Finalized (V2.0) |
| 3 | State Machine + Sequence + ER Diagrams | ✅ Finalized |
| 4 | HTML Prototypes (all states, all roles) | ✅ Complete |
| 5 | Global CSS (design system) | ✅ Complete |
| 6 | CC Field Reference | ✅ Complete |
| 7 | Database Design Document | ✅ Complete (V1.1 — PostgreSQL) |
| 8 | Backend Engineering Blueprint | ✅ Complete |
| 9 | REST API Development (Go) | ⏳ Not started |
| 10 | Frontend (Svelte) | ⏳ Not started |

> This repository holds **documentation and design artifacts only.** The backend (Go) and frontend (Svelte) implementations live in separate repositories.

## What's in this Repo

### 1. Business Requirements Document (BRD)

The single source of truth for what the system does. Covers:

- Executive summary, scope, success criteria
- User roles & personas (CC Owner, Approver, Viewer, Admin)
- Workflow & states (6 states, 8 transitions, rejection flows)
- Field-level permissions (50 fields)
- 25 user stories with acceptance criteria
- Functional requirements
- Full field definitions for all 50 fields (data types, validation, dropdowns, mandatory rules)
- Business rules, including the E-Signature ruleset (§8.8)
- Electronic signature requirements across all seven decision transitions
- UI/UX guidelines, non-functional requirements, integration, limitations, assumptions, appendices

📄 `EA_QMS_Change_Control_BRD_V1_1.docx`

### 2. CC Field Reference

Condensed working reference for all 50 fields — type, nullability, max length, valid values, mandatory-per-transition sets, the audit-tracked fields, and the canonical string values used in implementation. The practical companion to the BRD's Appendix D.

📄 `CC_Field_Reference.docx` / `CC_Field_Reference.md`

### 3. Security Matrix

Defines who can do what, where, and when. 50 fields × states × 4 roles, colour-coded (green = editable, red = read-only). Drives field-level permissions throughout the application.

📄 `Security_Matrix_V2_0.xlsx`

### 4. Diagrams

- **State Machine** — 6 states (Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, Cancelled), all transitions including rejection loops, and editable fields per state per role.
- **Sequence Diagram** — Full CC Owner ↔ System ↔ Approver interaction across all workflow phases, including electronic-signature exchanges and both rejection scenarios.
- **Entity Relationship Diagram** — The database schema: 6 tables and their relationships.

📄 `Flow_Chart.png` + `Flow_Chart_Code.txt` (Mermaid)
📄 `Sequence_Diagram.png` + `Sequence_Diagram_Code.txt` (Mermaid)
📄 `ER_Diagram.png` + `ER_Diagram_Code.txt` (Mermaid)

### 5. HTML Prototypes

Comprehensive UI prototypes covering every workflow state from every role's perspective. Built with semantic HTML and the global CSS design system, including electronic-signature modals and signature-history panels. The frontend will reuse this markup and styling directly.

**Authentication & onboarding:**
`login.html`, `forgot-password.html`, `reset-password.html`, `email-reset-password.html`

**Dashboards & list views:**
`dashboard-*.html`, `all-change-controls.html`, `my-change-controls*.html`, `approvals*.html`

**Change Control Form** (every state × every relevant role):

- `cc-form-initated-state.html` / `-approver-view.html` — Initiated
- `cc-form-pending-implementation-approval-*.html` — CC Owner & Approver
- `cc-form-in-implementation-*.html` — CC Owner & Approver
- `cc-form-pending-final-approval-*.html` — CC Owner & Approver
- `cc-form-closed.html`, `cc-form-cancelled.html` — terminal states

**Settings:**
`settings-admin.html` (user management + role editing), `settings-profile.html`

### 6. Global CSS

Flat enterprise design system. Design tokens (CSS custom properties), typography, spacing, components, form patterns, modals, and the signature-history table. Single source for all styling — every prototype imports it, and it ports to the frontend framework directly.

📄 `global.css`

### 7. Database Design Document

The physical schema for PostgreSQL. Covers the ER diagram, all 6 table definitions, foreign keys, indexes, CHECK/DEFAULT constraints, seed data, generated-value and concurrency handling, and security considerations. References the BRD rather than restating it.

📄 `EA_QMS_Change_Control_Database_Design_V1_1.docx`

### 8. Backend Engineering Blueprint

The architectural decisions and conventions for the Go implementation — stack, project structure, handler patterns, authentication and session model, transaction handling, the two documented concurrency hazards, and coding principles.

📄 `BACKEND_BLUEPRINT.md`

## Architecture Approach

**Database-first, with business logic in the application layer:**

- **Database** — structural safety: constraints, foreign keys, defaults, unique/generated identifiers, enum validation via `CHECK`. Guarantees that cannot be raced live here.
- **Go application layer** — all business logic: state transitions, field validation, mandatory-field checks, role-based permissions, electronic-signature verification, audit logging, and notifications. Multi-step operations run inside explicit transactions for atomicity.

This keeps the schema clean and the business rules testable, and pushes correctness guarantees into the database wherever the database can enforce them natively.

## Planned Tech Stack

| Layer | Technology |
|-------|-----------|
| Database | PostgreSQL |
| Migrations | goose |
| Query layer | sqlc (hand-written SQL, type-safe Go — no ORM) |
| Driver | lib/pq |
| Backend | Go (net/http + ServeMux, no framework) |
| Authentication | JWT access tokens + refresh tokens; argon2id password hashing |
| Identifiers | UUID surrogate keys + human-readable `CC-XXX` business keys |
| Frontend | Svelte (reusing existing HTML/CSS) |

## What's Next

None of these have started yet:

- **REST API Design** — endpoint catalog, request/response contracts, validation rules, error responses.
- **Backend Development (Go)** — the API built on the blueprint: goose migrations, sqlc queries, JWT + refresh-token auth, role/ownership/state authorization, transactional workflow transitions, audit + electronic signature capture.
- **API Testing** — happy paths, validation failures, permission checks, full workflow scenarios.
- **Frontend Development (Svelte)** — built on the existing HTML prototypes and `global.css`, wired to the REST API.
- **End-to-end testing & deployment.**

## Repo Map

```
.
├── EA_QMS_Change_Control_BRD_V1_1.docx               # Business Requirements Document
├── CC_Field_Reference.docx / .md                     # Condensed 50-field working reference
│
├── Security_Matrix_V2_0.xlsx                         # 50 fields × states × 4 roles
│
├── Flow_Chart.png / .txt                             # State machine diagram (Mermaid)
├── Sequence_Diagram.png / .txt                       # Sequence diagram (Mermaid)
├── ER_Diagram.png / .txt                             # Entity relationship diagram (Mermaid)
│
├── *.html                                            # HTML prototypes (auth, dashboards, lists, forms, settings)
├── global.css                                        # Design system / global styles
│
├── EA_QMS_Change_Control_Database_Design_V1_1.docx   # Database design (PostgreSQL)
└── BACKEND_BLUEPRINT.md                              # Backend architecture & conventions
```

All comprehensive documentation lives in this repo — every business rule, field definition, state transition, permission rule, and DB column definition is documented.

## Document Versions

| Document | Version |
|----------|---------|
| BRD | 1.1 |
| Security Matrix | 2.0 |
| Database Design | 1.1 |

---

*EA QMS — Change Control Module · Phase 1*
