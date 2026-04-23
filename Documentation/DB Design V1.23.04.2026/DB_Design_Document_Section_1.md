# EAMI QMS — Change Control Module

## Database Design Document

**Version:** 1.0
**Date:** 22 April 2026
**Author:** DB Design Phase
**Status:** In Progress
**Source BRD:** Version 1.20.04.2026 (Approved & Signed Off — 21 April 2026)

---

# SECTION 1 — Overview & Conventions

---

## 1.1 Purpose of This Document

This document defines the complete database design for the EAMI QMS Change Control module. It serves as the authoritative reference for database creation and is the direct input to the SQL Server scripts that will build the production schema.

**What this document covers:**

- Table definitions with all columns, data types, constraints, and defaults
- Entity relationships and foreign key design
- Index strategy for query performance
- Stored procedure specifications (3 only — narrow scope)
- Seed data for development and testing
- Security considerations at the database level

**What this document does NOT cover:**

- Business logic implementation — handled in the C# service layer (Option C hybrid architecture)
- API endpoint design — covered in the separate API Design Document (next phase)
- Frontend implementation — Angular development is a later phase
- Technical API logging — Serilog configuration happens during API development, not DB design

**Architecture context (Option C — Hybrid):**

The system follows a DB-first approach where tables are created in SQL Server, then scaffolded into EF Core models. Business logic lives in the C# service layer, not in stored procedures. The database provides structural integrity (constraints, foreign keys, defaults) as a safety net, while the service layer handles state transition logic, field validation, permission checks, and all other business rules.

```
Request Flow:

    API Request
        → Controller (validates HTTP request)
            → Service Layer (ALL business logic in C#)
                → EF Core DbContext (scaffolded from DB)
                    → SQL Server (tables, constraints, defaults)
                → AuditService (calls usp_InsertAuditLog)
                → EmailService (sends notifications)
        → API Response
```

**Stored procedures are limited to exactly 3:**

1. `usp_InsertAuditLog` — atomic audit log write, called by the .NET AuditService
2. `usp_CheckActiveRecordsForUser` — active record check for the role change edge case (BR-8.4.11)
3. `usp_GenerateCCID` — sequential CC-ID generation in CC-XXX format

Everything else — state transitions, mandatory field validation, date validation, role-based permission checks, email notification triggers, file upload handling — is implemented in C# service classes.

---

## 1.2 Database Engine & Version

| Property | Value |
|----------|-------|
| Database Engine | Microsoft SQL Server 2022 |
| Compatibility Level | 160 (SQL Server 2022) |
| Database Name | `EAMI_QMS_ChangeControl` |
| Collation | `SQL_Latin1_General_CP1_CI_AS` (case-insensitive, accent-sensitive) |
| Recovery Model | Full (recommended for production — supports point-in-time restore) |
| ORM | Entity Framework Core (DB-first — scaffolded from the database) |

**Why SQL Server 2022:** Confirmed technology stack. The database uses standard SQL Server features (IDENTITY columns, CHECK constraints, DATETIME2, VARBINARY(MAX)) and does not rely on any version-specific features beyond SQL Server 2016. SQL Server 2022 is selected for long-term support and compatibility with the latest EF Core tooling.

---

## 1.3 Naming Conventions

All database objects follow consistent naming conventions to ensure readability, EF Core compatibility, and team-wide consistency.

**Table Names:**

| Convention | Pattern | Example |
|------------|---------|---------|
| Style | PascalCase, plural | `Users`, `ChangeControls` |
| Rationale | Plural because a table represents a collection of entities. EF Core's default convention maps `Users` table to `User` entity class automatically. |

**Column Names:**

| Convention | Pattern | Example |
|------------|---------|---------|
| Style | PascalCase, no underscores | `ChangeTitle`, `CreatedOn`, `LastUpdatedById` |
| FK columns | Referenced entity name + `Id` suffix | `ChangeOwnerId`, `AssignedApproverId` |
| Boolean columns | `Is` prefix | `IsActive` |
| Timestamp columns | Action + `On` suffix | `CreatedOn`, `LastUpdatedOn`, `ImplementationApprovalOn` |
| Rationale | PascalCase maps directly to C# property names after EF Core scaffold, requiring zero manual renaming. |

**Primary Key Columns:**

| Convention | Pattern | Example |
|------------|---------|---------|
| Style | Always named `Id` | `Users.Id`, `ChangeControls.Id` |
| Type | `INT IDENTITY(1,1)` for most tables, `BIGINT IDENTITY(1,1)` for AuditLogs |
| Rationale | Surrogate integer PKs are simpler for FK joins, EF Core navigation properties, and index performance than natural/composite keys. |

**Constraint Names:**

| Object Type | Pattern | Example |
|-------------|---------|---------|
| Primary Key | `PK_{TableName}` | `PK_Users` |
| Foreign Key | `FK_{ChildTable}_{ParentTable}_{Column}` | `FK_ChangeControls_Users_ChangeOwnerId` |
| CHECK | `CK_{TableName}_{ColumnName}` | `CK_ChangeControls_CurrentState` |
| DEFAULT | `DF_{TableName}_{ColumnName}` | `DF_ChangeControls_CurrentState` |
| UNIQUE | `UQ_{TableName}_{ColumnName(s)}` | `UQ_ChangeControls_CcId` |

**Index Names:**

| Object Type | Pattern | Example |
|-------------|---------|---------|
| Unique index | `UQ_{TableName}_{Column(s)}` | `UQ_Users_Email` |
| Non-clustered | `IX_{TableName}_{Column(s)}` | `IX_ChangeControls_CurrentState` |
| Composite | `IX_{TableName}_{Col1}_{Col2}` | `IX_ChangeControls_ChangeOwnerId_CurrentState` |

**Stored Procedure Names:**

| Convention | Pattern | Example |
|------------|---------|---------|
| Style | `usp_` prefix + PascalCase action | `usp_InsertAuditLog` |
| Rationale | `usp_` prefix distinguishes user procedures from system procedures (`sp_`). Microsoft recommends against the `sp_` prefix as SQL Server checks the `master` database first for `sp_` procedures. |

---

## 1.4 Data Type Conventions

Each SQL Server data type was chosen to balance storage efficiency, precision, and EF Core compatibility.

**Text fields:**

| Data Type | Used For | Rationale |
|-----------|----------|-----------|
| `NVARCHAR(n)` | All user-facing text fields | Unicode support required — BRD references Arabic locale (EN/AR language toggle in kiosk receipts). NVARCHAR stores 2 bytes per character, supporting international characters. Lengths match BRD-specified maximums (200, 500, 2000 characters). |
| `NVARCHAR(MAX)` | AuditLogs.OldValue, AuditLogs.NewValue | Audit entries may capture full field values (up to 2000 chars) and must accommodate any future field length changes without schema alteration. |
| `VARCHAR(10)` | ChangeControls.CcId | CC-ID format is CC-XXX — ASCII-only, no Unicode needed. VARCHAR saves 1 byte per character vs NVARCHAR. |

**Date and time fields:**

| Data Type | Used For | Rationale |
|-----------|----------|-----------|
| `DATETIME2(7)` | All timestamp fields (CreatedOn, LastUpdatedOn, approval timestamps, ActualClosureDate, AuditLogs.Timestamp) | Maximum precision (100 nanoseconds). SQL Server 2022 best practice — supersedes DATETIME. Larger date range and better accuracy than DATETIME. Maps cleanly to C# `DateTime` / `DateTimeOffset`. |
| `DATE` | Date-only fields (ProposedImplementationDate, TargetClosureDate, ActualImplementationDate) | BRD specifies these as date pickers with no time component. DATE type stores only the date portion (3 bytes, range 0001-01-01 to 9999-12-31). No time component avoids timezone-related comparison issues. |
| `TIME(0)` | Time-only fields (ImplementationWindowStart, ImplementationWindowEnd) | BRD examples show "02:00", "04:00" — hour and minute only, no seconds needed. TIME(0) = no fractional seconds, stores in 3 bytes. |

**Numeric and boolean fields:**

| Data Type | Used For | Rationale |
|-----------|----------|-----------|
| `INT` | Primary keys (Users, ChangeControls, FileAttachments), foreign key references | 4 bytes, range up to 2.1 billion. More than sufficient for CC records and user counts in this system's lifetime. |
| `BIGINT` | AuditLogs.Id | 8 bytes, range up to 9.2 quintillion. Audit logs grow indefinitely (no purge per BR-8.7.7) and each business action generates multiple entries (e.g., one approval = 3 field-change entries + 1 state transition entry). BIGINT is future-proof. |
| `BIT` | Users.IsActive | Boolean flag for user active/inactive status. BIT stores as 1 bit (SQL Server packs up to 8 BIT columns into 1 byte). |
| `BIGINT` | FileAttachments.FileSize | File size in bytes. BIGINT accommodates files up to 9.2 EB. While current max is 10MB (10,485,760 bytes), INT would also suffice, but BIGINT is the conventional choice for file sizes and prevents any future issues if limits are raised. |

**Binary fields:**

| Data Type | Used For | Rationale |
|-----------|----------|-----------|
| `VARBINARY(MAX)` | FileAttachments.FileData | Stores file content as binary data directly in the database. MAX allows up to 2GB per value — well above the 10MB file size limit. User confirmed DB storage (BLOB) over file system storage. |

---

## 1.5 NULL Strategy

The NULL strategy is driven by the Save Draft requirement (FR-6.1.12): CC Owners can save a record at any time without triggering mandatory field validation. This means user-editable fields must accept NULL values at the database level, even if they are mandatory at submission time.

**NOT NULL (always populated — enforced at DB level):**

These are system-generated fields that the application always populates. There is no scenario where they are empty.

| Fields | Rationale |
|--------|-----------|
| All surrogate PKs (`Id`) | IDENTITY columns are always populated |
| `CcId` | Generated by `usp_GenerateCCID` at creation — never NULL |
| `CurrentState` | Defaults to 'Initiated' at creation — always has a value |
| `ChangeOwnerId` | Set to the creating user's ID at creation — always populated |
| `LastUpdatedById` | Set to the creating user's ID at creation, updated on every save |
| `CreatedOn`, `LastUpdatedOn` | Default to SYSUTCDATETIME() — always populated |
| `ImplementationApprovalStatus` | Defaults to 'Not Submitted' — always has a value |
| `FinalApprovalStatus` | Defaults to 'Not Submitted' — always has a value |
| All `Users` table columns | Admin must provide all values when creating a user. `IsActive` defaults to 1. |
| All `AuditLogs` non-nullable columns | Audit entries are always fully populated by `usp_InsertAuditLog` |

**NULL (empty until populated — validated in C# service layer):**

| Fields | Rationale |
|--------|-----------|
| All CC Owner-editable fields (ChangeTitle through Comments) | Save Draft allows partial entry. Mandatory validation happens at submission time in the C# service layer, not at the DB level. |
| Approver-editable fields (Decision, RiskLevel, DecisionComments, FinalDecision, FinalComments) | Not populated until the Approver submits their decision. |
| System-generated approval fields (ImplementationApprovalById, ImplementationApprovalOn, FinalApprovalById, FinalApprovalOn, ActualClosureDate) | Only populated when a specific approval action occurs. NULL until then. |
| `AssignedApproverId` | May not be assigned yet during draft state. |
| `CancellationReason` | Only populated if the CC is cancelled. NULL for all non-cancelled records. |
| `AuditLogs.FieldName` | NULL for non-field actions (e.g., record creation — see BRD Section 6.6.3) |
| `AuditLogs.OldValue`, `AuditLogs.NewValue` | NULL for creation events where there is no old/new field value |

**Why not use NOT NULL + empty string for text fields?**

Empty string (`''`) and NULL are semantically different. NULL means "not yet provided" — the user hasn't entered anything. Empty string means "the user explicitly entered nothing." The Save Draft scenario is clearly "not yet provided," so NULL is the correct representation. Additionally, CHECK constraints on dropdown fields (e.g., `ChangeType IN ('Application', 'Infrastructure', ...)`) would reject empty strings, requiring additional logic to handle the draft state. NULL is cleaner — it bypasses CHECK constraints naturally (NULL is neither equal to nor not equal to any value in a CHECK list).

---

## 1.6 Lookup Value Strategy

**Decision: CHECK constraints on columns — no separate lookup tables.**

All dropdown/enum values are enforced via CHECK constraints directly on the column. There are no `LookupTypes`, `LookupValues`, or similar reference tables.

**Rationale:**

- The dropdown values in this system are a closed, stable set defined in the approved BRD. They are not user-configurable and do not change at runtime.
- CHECK constraints are simpler — fewer tables, no FK joins needed to validate a dropdown selection, no seed data complexity.
- EF Core maps CHECK-constrained NVARCHAR columns cleanly to C# `string` properties. The service layer can define C# `enum` or constant classes for these values — they don't need to come from the database.
- If a value needs to be added in a future phase (e.g., adding "Emergency" to ChangeCategory), it's a simple `ALTER TABLE ... DROP/ADD CONSTRAINT` operation. This is a planned schema change, not a data entry.

**CHECK constraints are defined for 15 fields across 4 tables:** ChangeControls (11 fields), Users (1 field), FileAttachments (1 field), AuditLogs (2 fields). The complete list of all valid values is in Section 6.1.

**NULL and CHECK constraints:** A NULL value does not violate a CHECK constraint. SQL Server evaluates CHECK constraints using three-valued logic: a CHECK passes if the expression is TRUE or UNKNOWN (NULL). This means a user-editable dropdown field like `ChangeType` can be NULL (draft state) without conflicting with its CHECK constraint `IN ('Application', 'Infrastructure', ...)`. Only an explicit invalid value (e.g., `'InvalidType'`) would be rejected.

---

## 1.7 Design Principles

These principles govern all design decisions in this document. Each maps to a specific BRD requirement or confirmed architectural decision.

**DP-1: No Cascade Deletes — Indefinite Retention**

All foreign key relationships use `ON DELETE NO ACTION`. No record in any table is ever physically deleted. This directly implements the BRD data retention rules:

- CC records retained indefinitely, never deleted (BR-8.7.8)
- Audit records retained indefinitely, never deleted, never modified (BR-8.7.7)
- User records retained even after deactivation (BR-8.7.9)
- Cancelled CC records remain in the system permanently (FR-6.5.9)

User "deletion" is a soft-delete: setting `IsActive = 0`. The user record remains for audit trail and FK referential integrity.

**DP-2: Surrogate Primary Keys**

Every table uses an auto-incrementing integer (`INT IDENTITY(1,1)` or `BIGINT IDENTITY(1,1)`) as its primary key. Business identifiers like `CcId` (CC-XXX) and `Email` are UNIQUE constraints, not primary keys.

Rationale:
- Surrogate keys are stable — they never change, even if the business identifier format evolves
- INT joins are faster than NVARCHAR joins (4 bytes vs variable-length string comparison)
- EF Core navigation properties work cleanly with integer FKs
- The business key (`CcId`) remains accessible via its UNIQUE index for API lookups

**DP-3: Unicode Support (NVARCHAR)**

All user-facing text columns use `NVARCHAR` (Unicode) rather than `VARCHAR` (ASCII-only). The BRD references Arabic locale support (EN/AR language toggle on kiosk receipts), and user names, descriptions, and comments may contain international characters. The only exception is `CcId` which uses `VARCHAR(10)` — the CC-XXX format is inherently ASCII.

**DP-4: Validation at Two Levels**

The database enforces structural validity (data types, FK relationships, CHECK constraints, NOT NULL on system fields) as a safety net. The C# service layer enforces business validity (mandatory fields at submission, date validation rules, role-based permission checks, state transition logic). This two-level approach means:

- A bad value (e.g., `ChangeType = 'InvalidType'`) is rejected by the DB even if the service layer has a bug
- A valid but incomplete record (draft with NULL mandatory fields) is accepted by the DB because the service layer hasn't enforced submission rules yet
- Business rules like "Proposed Implementation Date must be ≥ 2 business days from today" are too complex for CHECK constraints and belong in C#

**DP-5: Audit Log Independence**

The AuditLogs table is structurally independent from the tables it audits. It uses string-based `EntityType` and `EntityId` (not FK references to ChangeControls or Users) for the audited entity. It does use an FK for `PerformedById` (who performed the action) because that relationship is direct and always valid. The `PerformedByName` column is deliberately denormalized — it captures the user's display name at the time of the action, not a reference to the current name. This ensures audit records remain accurate even if a user's name is later changed by Admin.

**DP-6: Single File Per Upload Field**

The BRD specifies two file upload fields (Supporting Documents — field #24, Implementation Evidence — field #34), each supporting a single file upload. The FileAttachments table enforces this with a UNIQUE constraint on `(ChangeControlId, FieldName)`. File replacement (uploading a new file to a field that already has one) is handled in the C# service layer by deleting the existing row and inserting a new one within a transaction.

**DP-7: Phase 1 Scope Only**

This design covers the Change Control module only. No tables, columns, or relationships are added for future modules (CAPA, Deviation, Risk Register). No fields are added beyond the 50 defined in the BRD. If future phases require schema changes, they will be managed through migration scripts — the current design is not pre-optimised for unknowns.

---

*End of Section 1*

---

**Section 1 Verification:**

- [x] Purpose and architectural context documented
- [x] SQL Server 2022 confirmed with database name
- [x] All naming conventions defined (tables, columns, PKs, FKs, constraints, indexes, SPs)
- [x] All data type choices documented with rationale
- [x] NULL strategy explained with Save Draft justification
- [x] Lookup strategy confirmed (CHECK constraints, no lookup tables)
- [x] All 7 design principles documented with BRD traceability
- [x] Option C hybrid architecture referenced — 3 SPs only, business logic in C#
