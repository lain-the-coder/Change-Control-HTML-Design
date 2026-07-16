# EA QMS — Change Control Module

**🔗 [Live prototype](https://lain-the-coder.github.io/Change-Control-HTML-Design/)** · **[Documentation](./Documentation)**

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

## Live Prototype

The full set of UI prototypes is hosted on **GitHub Pages**:

**→ https://lain-the-coder.github.io/Change-Control-HTML-Design/**

The landing page groups every screen by user role (CC Owner, Approver, Admin) and workflow state. It is a **static prototype** — navigation works between screens, but data does not persist and forms are illustrative. The substance of the project is the documentation behind it: the BRD, database design, and security matrix.

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

📁 `Documentation/BRD/`

### 2. CC Field Reference

Condensed working reference for all 50 fields — type, nullability, max length, valid values, mandatory-per-transition sets, the audit-tracked fields, and the canonical string values used in implementation. The practical companion to the BRD's Appendix D.

📁 `Documentation/BRD/`

### 3. Security Matrix

Defines who can do what, where, and when. 50 fields × states × 4 roles, colour-coded (green = editable, red = read-only). Drives field-level permissions throughout the application.

📄 `Documentation/Security Matrix V2.0.xlsx`

### 4. Diagrams

- **State Machine** — 6 states (Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, Cancelled), all transitions including rejection loops, and editable fields per state per role.
- **Sequence Diagram** — Full CC Owner ↔ System ↔ Approver interaction across all workflow phases, including electronic-signature exchanges and both rejection scenarios.
- **Entity Relationship Diagram** — The database schema: 6 tables and their relationships.

📁 `Documentation/Diagrams/` (Mermaid source + rendered images)

### 5. Database Design Document

The physical schema for PostgreSQL. Covers the ER diagram, all 6 table definitions, foreign keys, indexes, CHECK/DEFAULT constraints, seed data, generated-value and concurrency handling, and security considerations. References the BRD rather than restating it.

📁 `Documentation/DB/`

### 6. HTML Prototypes

Interactive UI prototypes covering every workflow state from every role's perspective, including electronic-signature modals and signature-history panels. Organised into three self-contained flow folders — each curated to its role, with its own copy of `global.css` and only the screens relevant to that role.

- **First-Flow-Admin-User** — user management (create users, edit roles, BR-8.4.11 role-change block), read-only CC view.
- **Second-Flow-End-User-Change-Control-Owner-Creator** — the full CC Owner workflow: every form state from Initiated through Closed/Cancelled.
- **Third-Flow-End-User-Approver** — the Approver's views, including the approvals queue and both decision gates.

📁 `HTML/`

### 7. Global CSS

Flat enterprise design system. Design tokens (CSS custom properties), typography, spacing, components, form patterns, modals, and the signature-history table. Every prototype imports it, and it ports to the frontend framework directly.

📄 `HTML/<flow>/global.css`

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
├── index.html                        # Prototype launcher (GitHub Pages landing)
├── README.md
│
├── Documentation/
│   ├── BRD/                          # Business Requirements Document (V1.1) + CC Field Reference
│   ├── DB/                           # Database Design Document (V1.1, PostgreSQL)
│   ├── Diagrams/                     # State machine, sequence, ER diagrams (Mermaid + images)
│   └── Security Matrix V2.0.xlsx     # 50 fields × states × 4 roles
│
└── HTML/                             # UI prototypes — curated per role, self-contained
    ├── First-Flow-Admin-User/                                 # Admin: user management, read-only CC view
    ├── Second-Flow-End-User-Change-Control-Owner-Creator/     # CC Owner: full workflow, every state
    └── Third-Flow-End-User-Approver/                          # Approver: decision gates, approvals queue
```

Each flow folder is self-contained — its own `global.css` and only the screens relevant to that role — so it can be opened and navigated independently.

## Document Versions

| Document | Version |
|----------|---------|
| BRD | 1.1 |
| Security Matrix | 2.0 |
| Database Design | 1.1 |

---

*EA QMS — Change Control Module · Phase 1*
