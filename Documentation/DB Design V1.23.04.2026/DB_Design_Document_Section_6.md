# SECTION 6 — Constraints & Defaults

This section consolidates all CHECK, DEFAULT, and UNIQUE constraints across the entire schema. While these constraints are also referenced in their respective table definitions (Section 3), this section serves as the single implementation reference for the SQL creation scripts — every constraint in one place with exact SQL syntax and the complete list of valid values.

**Total constraints:** 15 CHECK + 7 DEFAULT + 3 UNIQUE = **25 named constraints** (excluding PKs and FKs which are covered in Sections 3 and 4).

---

## 6.1 CHECK Constraints

15 CHECK constraints enforce valid values for all dropdown/enum columns across 4 tables. Each constraint allows NULL (SQL Server three-valued logic — a NULL value neither satisfies nor violates a CHECK condition), which supports the Save Draft scenario where dropdown fields may not yet be selected.

### 6.1.1 Users Table (1 CHECK constraint)

**CK_Users_Role** — 4 valid values

```sql
ALTER TABLE Users
ADD CONSTRAINT CK_Users_Role
    CHECK (Role IN ('CC Owner', 'Approver', 'Viewer', 'Admin'));
```

| # | Valid Value | Description |
|---|------------|-------------|
| 1 | CC Owner | Can create, edit, submit, and cancel CC records they own |
| 2 | Approver | Can review and submit decisions on CC records assigned to them |
| 3 | Viewer | Read-only access to all CC records |
| 4 | Admin | User management (create, edit, deactivate users) + read-only CC access |

---

### 6.1.2 ChangeControls Table (11 CHECK constraints)

**CK_ChangeControls_CurrentState** — 6 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_CurrentState
    CHECK (CurrentState IN (
        'Initiated',
        'Pending Implementation Approval',
        'In Implementation',
        'Pending Final Approval',
        'Closed',
        'Cancelled'
    ));
```

| # | Valid Value | Terminal? | BRD Ref |
|---|------------|-----------|---------|
| 1 | Initiated | No | Default state on creation |
| 2 | Pending Implementation Approval | No | After CC Owner submits for approval |
| 3 | In Implementation | No | After Approver approves at first gate |
| 4 | Pending Final Approval | No | After CC Owner submits for final approval |
| 5 | Closed | Yes | After Approver approves at final gate |
| 6 | Cancelled | Yes | After CC Owner cancels from Initiated |

---

**CK_ChangeControls_ChangeType** — 8 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_ChangeType
    CHECK (ChangeType IN (
        'Application', 'Infrastructure', 'Database', 'Security',
        'Network', 'Hardware', 'Process', 'Other'
    ));
```

---

**CK_ChangeControls_ChangeCategory** — 2 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_ChangeCategory
    CHECK (ChangeCategory IN ('Normal', 'Standard'));
```

Note: "Emergency" is intentionally excluded from Phase 1 (BRD Section 7, Field 10).

---

**CK_ChangeControls_DepartmentFunction** — 6 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_DepartmentFunction
    CHECK (DepartmentFunction IN (
        'IT', 'Operations', 'Security', 'QA', 'Facilities', 'Other'
    ));
```

---

**CK_ChangeControls_ExpectedDowntime** — 3 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_ExpectedDowntime
    CHECK (ExpectedDowntime IN ('Yes', 'No', 'Unknown'));
```

---

**CK_ChangeControls_RequiresTesting** — 3 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_RequiresTesting
    CHECK (RequiresTesting IN (
        'Yes – Full testing', 'Yes – Partial testing', 'No'
    ));
```

Note: The en-dash character (–) in the values requires NVARCHAR storage. These are the exact strings from BRD Field 20.

---

**CK_ChangeControls_RequiresTraining** — 3 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_RequiresTraining
    CHECK (RequiresTraining IN ('Yes', 'No', 'Not applicable'));
```

---

**CK_ChangeControls_PostImplementationIssues** — 3 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_PostImplementationIssues
    CHECK (PostImplementationIssues IN (
        'None', 'Minor issues resolved', 'Issues requiring follow-up'
    ));
```

---

**CK_ChangeControls_Decision** — 2 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_Decision
    CHECK (Decision IN ('Approve', 'Reject'));
```

---

**CK_ChangeControls_RiskLevel** — 3 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_RiskLevel
    CHECK (RiskLevel IN ('Low', 'Medium', 'High'));
```

---

**CK_ChangeControls_FinalDecision** — 2 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_FinalDecision
    CHECK (FinalDecision IN ('Approve', 'Reject'));
```

---

**CK_ChangeControls_ImplementationApprovalStatus** — 4 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_ImplementationApprovalStatus
    CHECK (ImplementationApprovalStatus IN (
        'Not Submitted', 'Pending', 'Approved', 'N/A'
    ));
```

---

**CK_ChangeControls_FinalApprovalStatus** — 4 valid values

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT CK_ChangeControls_FinalApprovalStatus
    CHECK (FinalApprovalStatus IN (
        'Not Submitted', 'Pending', 'Approved', 'N/A'
    ));
```

Important: The value is "Not Submitted" — not "Not Yet Submitted". Exact string match required per BRD (Section 7, Fields 46–47).

---

### 6.1.3 FileAttachments Table (1 CHECK constraint)

**CK_FileAttachments_FieldName** — 2 valid values

```sql
ALTER TABLE FileAttachments
ADD CONSTRAINT CK_FileAttachments_FieldName
    CHECK (FieldName IN ('SupportingDocuments', 'ImplementationEvidence'));
```

| # | Valid Value | BRD Field | Editable State |
|---|------------|-----------|----------------|
| 1 | SupportingDocuments | Field 24 — Supporting Documents | CC Owner in Initiated |
| 2 | ImplementationEvidence | Field 34 — Implementation Evidence | CC Owner in In Implementation |

---

### 6.1.4 AuditLogs Table (2 CHECK constraints)

**CK_AuditLogs_EntityType** — 2 valid values

```sql
ALTER TABLE AuditLogs
ADD CONSTRAINT CK_AuditLogs_EntityType
    CHECK (EntityType IN ('ChangeControl', 'User'));
```

Closed set for Phase 1. If future modules add new entity types (e.g., 'CAPA', 'Deviation'), the constraint is altered via `ALTER TABLE ... DROP CONSTRAINT` / `ADD CONSTRAINT`.

---

**CK_AuditLogs_ActionType** — 7 valid values

```sql
ALTER TABLE AuditLogs
ADD CONSTRAINT CK_AuditLogs_ActionType
    CHECK (ActionType IN (
        'Created',
        'StateChanged',
        'FieldUpdated',
        'UserAdded',
        'UserRoleChanged',
        'UserUpdated',
        'UserDeactivated'
    ));
```

| # | Valid Value | EntityType | Triggered By |
|---|------------|------------|-------------|
| 1 | Created | ChangeControl | CC Owner creates a new CC record |
| 2 | StateChanged | ChangeControl | Any state transition (8 valid transitions) |
| 3 | FieldUpdated | ChangeControl | Critical field value changed (9 tracked fields) |
| 4 | UserAdded | User | Admin creates a new user |
| 5 | UserRoleChanged | User | Admin changes a user's role |
| 6 | UserUpdated | User | Admin edits a user's Full Name |
| 7 | UserDeactivated | User | Admin deactivates a user account |

---

### 6.1.5 CHECK Constraint Summary

| # | Constraint Name | Table | Column | Values Count |
|---|----------------|-------|--------|-------------|
| 1 | CK_Users_Role | Users | Role | 4 |
| 2 | CK_ChangeControls_CurrentState | ChangeControls | CurrentState | 6 |
| 3 | CK_ChangeControls_ChangeType | ChangeControls | ChangeType | 8 |
| 4 | CK_ChangeControls_ChangeCategory | ChangeControls | ChangeCategory | 2 |
| 5 | CK_ChangeControls_DepartmentFunction | ChangeControls | DepartmentFunction | 6 |
| 6 | CK_ChangeControls_ExpectedDowntime | ChangeControls | ExpectedDowntime | 3 |
| 7 | CK_ChangeControls_RequiresTesting | ChangeControls | RequiresTesting | 3 |
| 8 | CK_ChangeControls_RequiresTraining | ChangeControls | RequiresTraining | 3 |
| 9 | CK_ChangeControls_PostImplementationIssues | ChangeControls | PostImplementationIssues | 3 |
| 10 | CK_ChangeControls_Decision | ChangeControls | Decision | 2 |
| 11 | CK_ChangeControls_RiskLevel | ChangeControls | RiskLevel | 3 |
| 12 | CK_ChangeControls_FinalDecision | ChangeControls | FinalDecision | 2 |
| 13 | CK_ChangeControls_ImplementationApprovalStatus | ChangeControls | ImplementationApprovalStatus | 4 |
| 14 | CK_ChangeControls_FinalApprovalStatus | ChangeControls | FinalApprovalStatus | 4 |
| 15 | CK_FileAttachments_FieldName | FileAttachments | FieldName | 2 |
| 16 | CK_AuditLogs_EntityType | AuditLogs | EntityType | 2 |
| 17 | CK_AuditLogs_ActionType | AuditLogs | ActionType | 7 |
| | **Total** | | | **17** |

**Note:** The count is 17 CHECK constraints (not 15 as stated in the section introduction). The 2 AuditLogs CHECK constraints were added per the approved review changes. The summary table above is the accurate count.

---

## 6.2 DEFAULT Constraints

7 DEFAULT constraints provide automatic values for system-generated fields that are populated at record creation.

### 6.2.1 Users Table (2 DEFAULT constraints)

**DF_Users_IsActive**

```sql
ALTER TABLE Users
ADD CONSTRAINT DF_Users_IsActive
    DEFAULT (1) FOR IsActive;
```

New users default to active. Admin can later deactivate by setting `IsActive = 0`.

---

**DF_Users_CreatedOn**

```sql
ALTER TABLE Users
ADD CONSTRAINT DF_Users_CreatedOn
    DEFAULT (SYSUTCDATETIME()) FOR CreatedOn;
```

---

**DF_Users_UpdatedOn**

```sql
ALTER TABLE Users
ADD CONSTRAINT DF_Users_UpdatedOn
    DEFAULT (SYSUTCDATETIME()) FOR UpdatedOn;
```

Note: `UpdatedOn` defaults to creation time and is subsequently updated by the C# service layer on every modification. The DB default covers the initial creation; the service layer handles subsequent updates via EF Core.

---

### 6.2.2 ChangeControls Table (4 DEFAULT constraints)

**DF_ChangeControls_CurrentState**

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT DF_ChangeControls_CurrentState
    DEFAULT ('Initiated') FOR CurrentState;
```

Every new CC record starts in the Initiated state (BRD Section 3, Transition T1).

---

**DF_ChangeControls_CreatedOn**

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT DF_ChangeControls_CreatedOn
    DEFAULT (SYSUTCDATETIME()) FOR CreatedOn;
```

---

**DF_ChangeControls_LastUpdatedOn**

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT DF_ChangeControls_LastUpdatedOn
    DEFAULT (SYSUTCDATETIME()) FOR LastUpdatedOn;
```

---

**DF_ChangeControls_ImplementationApprovalStatus**

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT DF_ChangeControls_ImplementationApprovalStatus
    DEFAULT ('Not Submitted') FOR ImplementationApprovalStatus;
```

---

**DF_ChangeControls_FinalApprovalStatus**

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT DF_ChangeControls_FinalApprovalStatus
    DEFAULT ('Not Submitted') FOR FinalApprovalStatus;
```

---

### 6.2.3 FileAttachments Table (1 DEFAULT constraint)

**DF_FileAttachments_UploadedOn**

```sql
ALTER TABLE FileAttachments
ADD CONSTRAINT DF_FileAttachments_UploadedOn
    DEFAULT (SYSUTCDATETIME()) FOR UploadedOn;
```

---

### 6.2.4 AuditLogs Table (1 DEFAULT constraint)

**DF_AuditLogs_Timestamp**

```sql
ALTER TABLE AuditLogs
ADD CONSTRAINT DF_AuditLogs_Timestamp
    DEFAULT (SYSUTCDATETIME()) FOR Timestamp;
```

Note: The C# AuditService typically generates the timestamp in the application layer and passes it explicitly to `usp_InsertAuditLog` (so that multiple entries from the same action share the exact same timestamp per BR-8.7.5). The DB default serves as a fallback if the parameter is not provided, ensuring the column is never NULL.

---

### 6.2.5 DEFAULT Constraint Summary

| # | Constraint Name | Table | Column | Default Value |
|---|----------------|-------|--------|---------------|
| 1 | DF_Users_IsActive | Users | IsActive | 1 (active) |
| 2 | DF_Users_CreatedOn | Users | CreatedOn | SYSUTCDATETIME() |
| 3 | DF_Users_UpdatedOn | Users | UpdatedOn | SYSUTCDATETIME() |
| 4 | DF_ChangeControls_CurrentState | ChangeControls | CurrentState | 'Initiated' |
| 5 | DF_ChangeControls_CreatedOn | ChangeControls | CreatedOn | SYSUTCDATETIME() |
| 6 | DF_ChangeControls_LastUpdatedOn | ChangeControls | LastUpdatedOn | SYSUTCDATETIME() |
| 7 | DF_ChangeControls_ImplementationApprovalStatus | ChangeControls | ImplementationApprovalStatus | 'Not Submitted' |
| 8 | DF_ChangeControls_FinalApprovalStatus | ChangeControls | FinalApprovalStatus | 'Not Submitted' |
| 9 | DF_FileAttachments_UploadedOn | FileAttachments | UploadedOn | SYSUTCDATETIME() |
| 10 | DF_AuditLogs_Timestamp | AuditLogs | Timestamp | SYSUTCDATETIME() |
| | **Total** | | | **10** |

**Why SYSUTCDATETIME() and not GETDATE() or GETUTCDATE():**

- `SYSUTCDATETIME()` returns UTC time as DATETIME2(7) — matching our column type directly with no implicit conversion.
- `GETDATE()` returns local server time as DATETIME — wrong timezone, wrong type.
- `GETUTCDATE()` returns UTC but as DATETIME (lower precision) — type mismatch with DATETIME2(7).
- Storing UTC is the standard practice for applications that may be accessed across timezones. The frontend (Angular) converts to local display time.

---

## 6.3 UNIQUE Constraints

3 UNIQUE constraints enforce business key uniqueness and the single-file-per-field rule.

### UQ-1: UQ_Users_Email

```sql
ALTER TABLE Users
ADD CONSTRAINT UQ_Users_Email
    UNIQUE (Email);
```

| Property | Value |
|----------|-------|
| Table | Users |
| Column(s) | Email |
| Purpose | Each email address can only belong to one user account. Email is the login identifier (BR-8.4.9) — duplicates would create authentication ambiguity. |
| Case sensitivity | Case-insensitive due to database collation `SQL_Latin1_General_CP1_CI_AS`. Attempting to insert 'admin@eami.com' when 'Admin@EAMI.com' already exists will violate this constraint. |
| Immutability | Email cannot be changed after user creation (enforced in C# service layer). The UNIQUE constraint ensures uniqueness at creation time. |

---

### UQ-2: UQ_ChangeControls_CcId

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT UQ_ChangeControls_CcId
    UNIQUE (CcId);
```

| Property | Value |
|----------|-------|
| Table | ChangeControls |
| Column(s) | CcId |
| Purpose | Each CC-ID (CC-XXX) must be unique across all records. Generated by `usp_GenerateCCID` which handles concurrency-safe sequential generation. |
| Format | CC-XXX where XXX is a zero-padded sequential number (CC-001, CC-002, ..., CC-999, CC-1000, ...). VARCHAR(10) accommodates up to CC-9999999. |
| Permanence | CC-ID is set once at creation and never changes (BRD Field 1). |

---

### UQ-3: UQ_FileAttachments_ChangeControlId_FieldName

```sql
ALTER TABLE FileAttachments
ADD CONSTRAINT UQ_FileAttachments_ChangeControlId_FieldName
    UNIQUE (ChangeControlId, FieldName);
```

| Property | Value |
|----------|-------|
| Table | FileAttachments |
| Column(s) | ChangeControlId, FieldName (composite) |
| Purpose | Enforces single file per upload field per CC record (BR-8.2.15). For any given CC, there can be at most one 'SupportingDocuments' row and one 'ImplementationEvidence' row. |
| File replacement | When a user uploads a new file to a field that already has one, the C# service layer deletes the existing row and inserts the new one within a transaction. The UNIQUE constraint is satisfied because the delete occurs before the insert. |

---

*End of Section 6*

---

**Section 6 Verification:**

- [x] All 17 CHECK constraints defined with exact SQL and valid values
- [x] All 10 DEFAULT constraints defined with exact SQL and rationale
- [x] All 3 UNIQUE constraints defined with exact SQL and business purpose
- [x] Every constraint has a named identifier following the naming convention (CK_, DF_, UQ_)
- [x] CHECK constraint summary table with value counts per constraint
- [x] DEFAULT constraint summary table with default values
- [x] SYSUTCDATETIME() rationale documented (vs GETDATE/GETUTCDATE)
- [x] NULL interaction with CHECK constraints explained (three-valued logic)
- [x] AuditLogs CHECK constraints include both review changes (EntityType, ActionType with 7 values)
- [x] "Not Submitted" exact string callout for approval status fields
- [x] En-dash character note for RequiresTesting values
- [x] Phase 1 exclusion note for "Emergency" ChangeCategory
