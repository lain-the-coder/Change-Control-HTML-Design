# SECTION 3 — Table Definitions (Part 1 of 3)

This section defines every table in the database with complete column specifications. Each column includes its data type, nullability, default value, constraints, and traceability to the BRD field definition.

---

## 3.1 Users Table

**Purpose:** Stores user accounts for authentication and role-based access control. Each user has exactly one role at any time. Users are never physically deleted — deactivation sets `IsActive = 0` while retaining the record for audit trail and FK referential integrity.

**Total columns:** 8
**Primary key:** `Id` (INT IDENTITY)
**BRD references:** Section 2 (User Roles & Personas), Section 8.4.9 (Manage Users), Section 8.4.11 (Role Change Restriction), Section 8.7.9 (User Record Retention)

| # | Column | Data Type | Nullable | Default | Constraints | Description |
|---|--------|-----------|----------|---------|-------------|-------------|
| 1 | `Id` | INT IDENTITY(1,1) | NOT NULL | Auto-increment | PK_Users | Surrogate primary key. Auto-generated, never modified. |
| 2 | `FullName` | NVARCHAR(200) | NOT NULL | — | — | User's display name. Set by Admin at creation. Editable by Admin (BR-8.4.9). Displayed in CC meta-grid as "Change Owner" and "Last Updated By" values. NVARCHAR for Unicode/Arabic character support. |
| 3 | `Email` | NVARCHAR(256) | NOT NULL | — | UQ_Users_Email | Login identifier. Set at creation, **immutable after creation** — cannot be changed by Admin or any user (BR-8.4.9). NVARCHAR(256) per RFC 5321 maximum email length. Case-insensitive matching handled in C# service layer. |
| 4 | `PasswordHash` | NVARCHAR(500) | NOT NULL | — | — | BCrypt hash of the user's password. **Never stored as plain text.** NVARCHAR(500) accommodates BCrypt output (typically 60 characters) with room for future algorithm changes. Password cannot be reset via Admin UI — users use the Forgot Password flow, or passwords are managed at the database level (BR-8.4.9). |
| 5 | `Role` | NVARCHAR(20) | NOT NULL | — | CK_Users_Role | The user's current role. Exactly one role per user at any time. Editable by Admin, subject to the active record restriction (BR-8.4.11). |
| 6 | `IsActive` | BIT | NOT NULL | 1 | DF_Users_IsActive | Active/inactive flag. `1` = active (can log in), `0` = deactivated (cannot log in). Deactivated users are retained in the database for audit trail referencing (BR-8.7.9). |
| 7 | `CreatedOn` | DATETIME2(7) | NOT NULL | SYSUTCDATETIME() | DF_Users_CreatedOn | Timestamp of when the user account was created by Admin. Set once, never changes. UTC time. |
| 8 | `UpdatedOn` | DATETIME2(7) | NOT NULL | SYSUTCDATETIME() | DF_Users_UpdatedOn | Timestamp of the last modification to this user record (name change, role change, deactivation). Updated by the C# service layer on every edit. UTC time. |

**CHECK constraint — CK_Users_Role:**

```sql
CHECK (Role IN ('CC Owner', 'Approver', 'Viewer', 'Admin'))
```

**Business rules enforced by the C# service layer (not the database):**

- Admin cannot change a user's role if the user has active CC records (BR-8.4.11) — supported by `usp_CheckActiveRecordsForUser`
- Email uniqueness is case-insensitive (e.g., `admin@eami.com` and `Admin@EAMI.com` are treated as the same) — the UNIQUE constraint handles exact-match uniqueness; the collation (`SQL_Latin1_General_CP1_CI_AS`, case-insensitive) ensures case-insensitive comparison
- Only Admin role can create, edit, or deactivate users
- Deactivated users cannot log in — checked during authentication in the service layer

---

## 3.2 ChangeControls Table

**Purpose:** The main Change Control record table. Contains all 50 BRD fields mapped to 48 columns (fields #24 and #34 are file uploads stored in the FileAttachments table, plus 1 surrogate PK column = 49 total columns).

**Total columns:** 49
**Primary key:** `Id` (INT IDENTITY)
**Business key:** `CcId` (VARCHAR(10), UNIQUE) — the CC-XXX identifier visible to users
**Foreign keys:** 5 (all referencing Users.Id)
**CHECK constraints:** 11 (on all dropdown/enum fields)
**BRD references:** Section 7 (all 3 parts — Field Definitions), Section 4 (Security Matrix), Section 8 (Business Rules)

Columns are grouped by BRD section to maintain traceability.

---

### 3.2.1 Identification (BRD Fields 1–6) — System-Generated

All 6 fields in this group are system-managed and read-only for all users in all states. They are populated automatically by the system at creation or on every save/submission.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 1 | `Id` | INT IDENTITY(1,1) | NOT NULL | Auto-increment | PK_ChangeControls | — | Surrogate PK (not a BRD field) |
| 2 | `CcId` | VARCHAR(10) | NOT NULL | — | UQ_ChangeControls_CcId | 1 | CC-ID |
| 3 | `CurrentState` | NVARCHAR(50) | NOT NULL | 'Initiated' | CK_ChangeControls_CurrentState, DF_ChangeControls_CurrentState | 2 | Current State |
| 4 | `ChangeOwnerId` | INT | NOT NULL | — | FK_ChangeControls_Users_ChangeOwnerId | 3 | Change Owner |
| 5 | `LastUpdatedById` | INT | NOT NULL | — | FK_ChangeControls_Users_LastUpdatedById | 4 | Last Updated By |
| 6 | `CreatedOn` | DATETIME2(7) | NOT NULL | SYSUTCDATETIME() | DF_ChangeControls_CreatedOn | 5 | Created On |
| 7 | `LastUpdatedOn` | DATETIME2(7) | NOT NULL | SYSUTCDATETIME() | DF_ChangeControls_LastUpdatedOn | 6 | Last Updated On |

**Column notes:**

- **CcId (VARCHAR, not NVARCHAR):** The CC-XXX format is inherently ASCII. VARCHAR saves 1 byte per character. Generated by `usp_GenerateCCID` stored procedure at record creation. Format: `CC-001`, `CC-002`, ..., `CC-999`, continuing beyond 3 digits if needed. Unique across all records, sequential, no gaps required but no duplicates permitted (BRD Field 1).
- **CurrentState:** Contains one of exactly 6 valid state values. Default is 'Initiated' on creation. Updated by the C# service layer during state transitions — never edited directly by a user.
- **ChangeOwnerId:** Set to the creating user's `Users.Id` at creation. Immutable — cannot be changed after creation. The user's full name is resolved via JOIN for display purposes.
- **LastUpdatedById:** Set to the creating user's ID at creation, then updated to the acting user's ID on every save, submission, or workflow action.
- **CreatedOn:** Immutable — set once at creation, never changes (BRD Field 5).
- **LastUpdatedOn:** Updated on every modification. Always ≥ CreatedOn (BRD Field 6).

**CHECK constraint — CK_ChangeControls_CurrentState:**

```sql
CHECK (CurrentState IN (
    'Initiated',
    'Pending Implementation Approval',
    'In Implementation',
    'Pending Final Approval',
    'Closed',
    'Cancelled'
))
```

---

### 3.2.2 Change Definition (BRD Fields 7–12) — CC Owner Editable in Initiated State

All 6 fields capture the core details of the proposed change. Editable by the CC Owner in the Initiated state only. Read-only in all other states for all roles.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 8 | `ChangeTitle` | NVARCHAR(200) | NULL | — | — | 7 | Change Title |
| 9 | `ChangeDescription` | NVARCHAR(2000) | NULL | — | — | 8 | Change Description |
| 10 | `ChangeType` | NVARCHAR(50) | NULL | — | CK_ChangeControls_ChangeType | 9 | Change Type |
| 11 | `ChangeCategory` | NVARCHAR(20) | NULL | — | CK_ChangeControls_ChangeCategory | 10 | Change Category |
| 12 | `DepartmentFunction` | NVARCHAR(50) | NULL | — | CK_ChangeControls_DepartmentFunction | 11 | Department / Function |
| 13 | `AffectedSystemsModules` | NVARCHAR(500) | NULL | — | — | 12 | Affected Systems / Modules |

**Column notes:**

- **ChangeTitle:** Single-line text input, max 200 characters. Mandatory at submission (BR-8.2.1), but NULL allowed for Save Draft.
- **ChangeDescription:** Multi-line textarea, max 2000 characters. Mandatory at submission.
- **ChangeType:** Dropdown with 8 options. NULL = no selection yet (draft state). The CHECK constraint allows NULL (SQL Server three-valued logic) but rejects any value not in the defined list.
- **ChangeCategory:** Dropdown with 2 options. "Emergency" is intentionally excluded from Phase 1 (BRD Section 7, Field 10 note).
- **DepartmentFunction:** Column name maps BRD field name "Department / Function" — the slash is removed for SQL compatibility.
- **AffectedSystemsModules:** Free-text field, max 500 characters. Example: "Kiosk App, Payment Service, Production Environment".

**CHECK constraint — CK_ChangeControls_ChangeType:**

```sql
CHECK (ChangeType IN (
    'Application', 'Infrastructure', 'Database', 'Security',
    'Network', 'Hardware', 'Process', 'Other'
))
```

**CHECK constraint — CK_ChangeControls_ChangeCategory:**

```sql
CHECK (ChangeCategory IN ('Normal', 'Standard'))
```

**CHECK constraint — CK_ChangeControls_DepartmentFunction:**

```sql
CHECK (DepartmentFunction IN (
    'IT', 'Operations', 'Security', 'QA', 'Facilities', 'Other'
))
```

---

### 3.2.3 Planning (BRD Fields 13–16) — CC Owner Editable in Initiated State

The Planning section captures the timeline for the proposed change. Two date fields with business day validation rules (enforced in C#) and two optional time fields for the implementation window.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 14 | `ProposedImplementationDate` | DATE | NULL | — | — | 13 | Proposed Implementation Date |
| 15 | `TargetClosureDate` | DATE | NULL | — | — | 14 | Target Closure Date |
| 16 | `ImplementationWindowStart` | TIME(0) | NULL | — | — | 15 | Implementation Window Start |
| 17 | `ImplementationWindowEnd` | TIME(0) | NULL | — | — | 16 | Implementation Window End |

**Column notes:**

- **ProposedImplementationDate:** DATE type (no time component). Mandatory at submission. Validation at submission time: must be ≥ 2 business days from current date AND must be in the future (BR-8.2.7, BR-8.2.9). All validation in C# service layer — no CHECK constraint on dates.
- **TargetClosureDate:** DATE type. Mandatory at submission. Validation: must be ≥ 10 business days from current date AND must be in the future (BR-8.2.8, BR-8.2.9). Editable when record returns to Initiated after rejection. Changes to this field are tracked in the audit log (BR-8.7.2).
- **ImplementationWindowStart / End:** TIME(0) — hour and minute only, no seconds. Both optional (BRD Fields 15–16). No cross-field validation between start and end in Phase 1.
- **Date re-validation on resubmission:** When a CC is rejected and returns to Initiated, date validations apply fresh based on the current date at resubmission time (BR-8.2.11). Dates valid at the original submission may no longer be valid.

---

### 3.2.4 Impact & Risk Assessment (BRD Fields 17–24) — CC Owner Editable in Initiated State

Eight fields capturing business justification, risk analysis, and supporting documentation. Note: Field 24 (Supporting Documents) is a file upload stored in the FileAttachments table, not in this table.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 18 | `ReasonForChange` | NVARCHAR(2000) | NULL | — | — | 17 | Reason for Change |
| 19 | `BusinessImpact` | NVARCHAR(2000) | NULL | — | — | 18 | Business Impact |
| 20 | `ExpectedDowntime` | NVARCHAR(20) | NULL | — | CK_ChangeControls_ExpectedDowntime | 19 | Expected Downtime |
| 21 | `RequiresTesting` | NVARCHAR(50) | NULL | — | CK_ChangeControls_RequiresTesting | 20 | Requires Testing |
| 22 | `RequiresTraining` | NVARCHAR(30) | NULL | — | CK_ChangeControls_RequiresTraining | 21 | Requires Training |
| 23 | `RiskRationale` | NVARCHAR(2000) | NULL | — | — | 22 | Risk Rationale |
| 24 | `KeyRisksMitigations` | NVARCHAR(2000) | NULL | — | — | 23 | Key Risks & Mitigations |
| — | *(Not in this table)* | — | — | — | — | 24 | Supporting Documents → FileAttachments |

**Column notes:**

- **ReasonForChange, BusinessImpact, RiskRationale, KeyRisksMitigations:** All multi-line textareas with 2000-character max (BR-8.2.12). All mandatory at submission. These are **non-critical fields** — edits to these fields are NOT tracked in the audit log (BR-8.7.3).
- **ExpectedDowntime:** 3-option dropdown. NVARCHAR(20) accommodates the longest value ('Unknown' = 7 chars) with room.
- **RequiresTesting:** 3-option dropdown. NVARCHAR(50) accommodates 'Yes – Full testing' (19 chars) and 'Yes – Partial testing' (21 chars). Note the en-dash character (–) in the values, requiring NVARCHAR.
- **RequiresTraining:** 3-option dropdown. NVARCHAR(30) accommodates 'Not applicable' (14 chars).
- **Field 24 — Supporting Documents:** This is a file upload field (optional, single file, max 10MB). The file data is stored in the FileAttachments table with `FieldName = 'SupportingDocuments'`. See Section 3.3.

**CHECK constraint — CK_ChangeControls_ExpectedDowntime:**

```sql
CHECK (ExpectedDowntime IN ('Yes', 'No', 'Unknown'))
```

**CHECK constraint — CK_ChangeControls_RequiresTesting:**

```sql
CHECK (RequiresTesting IN ('Yes – Full testing', 'Yes – Partial testing', 'No'))
```

**CHECK constraint — CK_ChangeControls_RequiresTraining:**

```sql
CHECK (RequiresTraining IN ('Yes', 'No', 'Not applicable'))
```

---

### 3.2.5 Implementation Plan & Validation (BRD Fields 25–28) — CC Owner Editable in Initiated State

Four textarea fields capturing the planned approach to implementing the change.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 25 | `HighLevelImplementationPlan` | NVARCHAR(2000) | NULL | — | — | 25 | High-Level Implementation Plan |
| 26 | `ValidationApproach` | NVARCHAR(2000) | NULL | — | — | 26 | Validation Approach |
| 27 | `SuccessCriteria` | NVARCHAR(2000) | NULL | — | — | 27 | Success Criteria |
| 28 | `RollbackBackoutPlan` | NVARCHAR(2000) | NULL | — | — | 28 | Rollback / Backout Plan |

**Column notes:**

- All four fields are multi-line textareas with 2000-character max (BR-8.2.12).
- All four are mandatory at submission for approval (BR-8.2.1).
- All four are **non-critical fields** — edits are NOT tracked in the audit log (BR-8.7.3).
- Column names use PascalCase with no special characters. BRD field name "Rollback / Backout Plan" maps to `RollbackBackoutPlan`.
- These fields are displayed as "Not applicable — Available after approval" when viewed in states other than Initiated (but only in the UI — the DB column simply stores the value regardless of state).

---

*End of Section 3 — Part 1 of 3*

**Part 1 covers:** Users table (8 columns) + ChangeControls fields 1–28 (28 columns including surrogate PK, noting field 24 goes to FileAttachments).

**Coming in Part 2:** ChangeControls fields 29–50 (remaining 21 columns) + Column Count Verification.

**Running column count for ChangeControls:**

| Group | Columns in Part 1 | Count |
|-------|-------------------|-------|
| Surrogate PK | Id | 1 |
| Identification (Fields 1–6) | CcId through LastUpdatedOn | 6 |
| Change Definition (Fields 7–12) | ChangeTitle through AffectedSystemsModules | 6 |
| Planning (Fields 13–16) | ProposedImplementationDate through ImplementationWindowEnd | 4 |
| Impact & Risk (Fields 17–23) | ReasonForChange through KeyRisksMitigations | 7 |
| Field 24 (Supporting Documents) | → FileAttachments | 0 |
| Impl Plan & Validation (Fields 25–28) | HighLevelImplementationPlan through RollbackBackoutPlan | 4 |
| **Part 1 subtotal** | | **28** |
| **Remaining in Part 2** | Fields 29–50 | **21** |
| **Expected total** | | **49** |
