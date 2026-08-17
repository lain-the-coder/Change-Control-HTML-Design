# EA QMS — Change Control Module

**🔗 [Live prototypes](https://lain-the-coder.github.io/Change-Control-HTML-Design/)** · **[Documentation](./Documentation)**
**⚙️ [Backend API](https://github.com/lain-the-coder/ea-qms-backend)** · **[API reference](https://lain-the-coder.github.io/ea-qms-backend/)**

A digital Change Control module for a Quality Management System. Replaces paper-based change control with a workflow-driven web application that enforces role-based permissions, captures a full audit trail, requires electronic signatures on every decision, and routes every change through a defined approval lifecycle.

This repository holds the **specification and design artifacts** — the business requirements, security matrix, database design, diagrams and UI prototypes. The Go backend lives in [ea-qms-backend](https://github.com/lain-the-coder/ea-qms-backend); the Svelte frontend will live in its own repository.

## Project Status

**Specification complete. Backend complete. Frontend not started.**

| # | Phase | Status |
|---|-------|--------|
| 1 | Business Requirements Document (BRD) | ✅ Complete (V1.2) |
| 2 | Security Matrix | ✅ Finalized (V2.1) |
| 3 | State Machine + Sequence + ER Diagrams | ✅ Finalized |
| 4 | HTML Prototypes (all states, all roles) | ✅ Complete |
| 5 | Global CSS (design system) | ✅ Complete |
| 6 | CC Field Reference | ✅ Complete (V1.2) |
| 7 | Database Design Document | ✅ Complete (V1.2 — PostgreSQL) |
| 8 | REST API Design | ✅ Complete (23 endpoints) |
| 9 | Backend Development (Go) | ✅ **Complete** — [ea-qms-backend](https://github.com/lain-the-coder/ea-qms-backend) |
| 10 | Frontend (Svelte) | ⏳ Not started |

The documents in this repository were **amended at backend completion** so that they describe what was built rather than what was intended — eleven amendments across four documents, each recorded with its reason. Where the implementation departed from the original specification, the specification was corrected rather than the deviation left undocumented.

## Live Prototype

The full set of UI prototypes is hosted on **GitHub Pages**:

**→ https://lain-the-coder.github.io/Change-Control-HTML-Design/**

The landing page groups every screen by user role (CC Owner, Approver, Admin) and workflow state. It is a **static prototype** — navigation works between screens, but data does not persist and forms are illustrative. The substance of the project is the documentation behind it: the BRD, database design, and security matrix.

## The Backend

**→ https://github.com/lain-the-coder/ea-qms-backend**

Go, PostgreSQL, `net/http` with no framework, sqlc for type-safe queries and no ORM. 23 endpoints covering authentication, user management, the change control lifecycle, all eight workflow transitions with electronic signatures, file attachments, and a dashboard.

The API reference is published and browsable:

**→ https://lain-the-coder.github.io/ea-qms-backend/**

That repository also carries a **Go coding guide** — 18 sections and 114 rules, each with code examples — and a decision log recording every choice made during the build, including the ones that reversed earlier decisions.

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

**V1.2** records the deliberate departures made during the build — narrowed file types, an all-or-nothing user update, a corrected session window, and five new Phase 1 limitations.

📁 `Documentation/BRD/`

### 2. CC Field Reference

Condensed working reference for all 50 fields — type, nullability, max length, valid values, mandatory-per-transition sets, the audit-tracked fields, and the canonical string values used in implementation. The practical companion to the BRD's Appendix D.

**V1.2** adds the complete audit scope — only nine of the fifty fields are audited — and marks the fields that were descoped.

⚠️ Its **Canonical String Values** section overrides the BRD on six enum strings that the BRD renders with en-dashes and the database requires with ASCII hyphens. This is the most likely source of silent failure in any client.

📁 `Documentation/BRD/`

### 3. Security Matrix

Defines who can do what, where, and when. 50 fields × states × 4 roles, colour-coded (green = editable, red = read-only). Drives field-level permissions throughout the application.

📄 `Documentation/Security Matrix V2.1.xlsx`

### 4. Diagrams

- **State Machine** — 6 states (Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, Cancelled), all transitions including rejection loops, and editable fields per state per role.
- **Sequence Diagram** — Full CC Owner ↔ System ↔ Approver interaction across all workflow phases, including electronic-signature exchanges and both rejection scenarios.
- **Entity Relationship Diagram** — The database schema: 6 tables and their relationships.

📁 `Documentation/Diagrams/` (Mermaid source + rendered images)

### 5. Database Design Document

The physical schema for PostgreSQL. Covers the ER diagram, all 6 table definitions, foreign keys, indexes, CHECK/DEFAULT constraints, seed data, generated-value and concurrency handling, and security considerations. References the BRD rather than restating it.

**V1.2** corrects the `change_controls` column count and records an index added during the build.

📁 `Documentation/DB/`

### 6. HTML Prototypes

Interactive UI prototypes covering every workflow state from every role's perspective, including electronic-signature modals and signature-history panels. Organised into three self-contained flow folders — each curated to its role, with its own copy of `global.css` and only the screens relevant to that role.

- **First-Flow-Admin-User** — user management (create users, edit roles, BR-8.4.11 role-change block), read-only CC view.
- **Second-Flow-End-User-Change-Control-Owner-Creator** — the full CC Owner workflow: every form state from Initiated through Closed/Cancelled.
- **Third-Flow-End-User-Approver** — the Approver's views, including the approvals queue and both decision gates.

Amended at backend completion to match the built API — enum values corrected to ASCII hyphens, the signature modal relabelled from Username to Email, the descoped upload field removed, and the profile screens made read-only.

📁 `HTML/`

### 7. Global CSS

Flat enterprise design system. Design tokens (CSS custom properties), typography, spacing, components, form patterns, modals, and the signature-history table. Every prototype imports it, and it ports to the frontend framework directly.

📄 `HTML/<flow>/global.css`

## Architecture Approach

**Database-first, with business logic in the application layer:**

- **Database** — structural safety: constraints, foreign keys, defaults, unique/generated identifiers, enum validation via `CHECK`. Guarantees that cannot be raced live here.
- **Go application layer** — all business logic: state transitions, field validation, mandatory-field checks, role-based permissions, electronic-signature verification, audit logging, and notifications. Multi-step operations run inside explicit transactions for atomicity.

This keeps the schema clean and the business rules testable, and pushes correctness guarantees into the database wherever the database can enforce them natively.

The reasoning behind each of those decisions — and forty-odd others — is recorded in the backend repository's [coding guide](https://github.com/lain-the-coder/ea-qms-backend/blob/main/GO_CODING_GUIDE.md).

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Database | PostgreSQL 14 |
| Migrations | goose |
| Query layer | sqlc (hand-written SQL, type-safe Go — no ORM) |
| Driver | lib/pq |
| Backend | Go (net/http + ServeMux, no framework) |
| Authentication | JWT access tokens + refresh tokens; argon2id password hashing |
| Identifiers | UUID surrogate keys + human-readable `CC-XXX` business keys |
| API documentation | OpenAPI 3.0.3 + Swagger UI |
| Frontend | Svelte 5 + SvelteKit in SPA mode, TypeScript (planned) |

## What's Next

- **Frontend Development (Svelte 5)** — built on the existing HTML prototypes and `global.css`, wired to the REST API. The plan is in the backend repository's [frontend blueprint](https://github.com/lain-the-coder/ea-qms-backend/blob/main/FRONTEND_BLUEPRINT.md).
- **End-to-end testing.**
- **Deployment.**

Deferred to a later release and recorded as such: password reset and change, self-service profile editing, email notifications, saved searches and reporting, and reassigning a change control to a different approver.

## Repo Map

```
.
├── index.html                        # Prototype launcher (GitHub Pages landing)
├── README.md
│
├── Documentation/
│   ├── BRD/                          # Business Requirements Document (V1.2) + CC Field Reference (V1.2)
│   ├── DB/                           # Database Design Document (V1.2, PostgreSQL)
│   ├── Diagrams/                     # State machine, sequence, ER diagrams (Mermaid + images)
│   └── Security Matrix V2.1.xlsx     # 50 fields × states × 4 roles
│
└── HTML/                             # UI prototypes — curated per role, self-contained
    ├── First-Flow-Admin-User/                                 # Admin: user management, read-only CC view
    ├── Second-Flow-End-User-Change-Control-Owner-Creator/     # CC Owner: full workflow, every state
    └── Third-Flow-End-User-Approver/                          # Approver: decision gates, approvals queue
```

Each flow folder is self-contained — its own `global.css` and only the screens relevant to that role — so it can be opened and navigated independently.

## Document Versions

| Document | Version | Last amended |
|----------|---------|--------------|
| BRD | 1.2 | Backend completion |
| Security Matrix | 2.1 | Backend completion |
| Database Design | 1.2 | Backend completion |
| CC Field Reference | 1.2 | Backend completion |

## Related repositories

| | |
|---|---|
| **[ea-qms-backend](https://github.com/lain-the-coder/ea-qms-backend)** | The Go API — 23 endpoints, complete |
| *Frontend* | Svelte 5, not yet started |

---

*EA QMS — Change Control Module · Phase 1*
