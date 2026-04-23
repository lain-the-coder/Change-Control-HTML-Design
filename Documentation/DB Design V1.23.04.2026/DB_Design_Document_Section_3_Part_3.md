# SECTION 3 — Table Definitions (Part 3 of 3)

Continuing from Part 2. This part covers the FileAttachments and AuditLogs tables.

---

## 3.3 FileAttachments Table

**Purpose:** Stores uploaded file content for the two file upload fields in the Change Control form: Supporting Documents (BRD Field 24) and Implementation Evidence (BRD Field 34). Files are stored as binary data (BLOB) directly in the database. Each upload field supports a single file per CC record, enforced by a UNIQUE constraint.

**Total columns:** 9
**Primary key:** `Id` (INT IDENTITY)
**Foreign keys:** 2 (ChangeControlId → ChangeControls.Id, UploadedById → Users.Id)
**BRD references:** Fields 24 and 34, Section 8.2.4 (File Upload Validation — BR-8.2.13 through BR-8.2.15), Section 6.3.6–6.3.9 (Implementation Evidence Upload)

| # | Column | Data Type | Nullable | Default | Constraints | Description |
|---|--------|-----------|----------|---------|-------------|-------------|
| 1 | `Id` | INT IDENTITY(1,1) | NOT NULL | Auto-increment | PK_FileAttachments | Surrogate primary key. |
| 2 | `ChangeControlId` | INT | NOT NULL | — | FK_FileAttachments_ChangeControls_ChangeControlId | The parent CC record this file belongs to. |
| 3 | `FieldName` | NVARCHAR(50) | NOT NULL | — | CK_FileAttachments_FieldName | Identifies which upload field this file is for. Restricted to exactly 2 values. |
| 4 | `FileName` | NVARCHAR(255) | NOT NULL | — | — | Original filename as uploaded by the user (e.g., "Impact_Assessment.pdf", "UAT_Scripts.pdf"). Displayed in the UI when the file is shown as read-only in subsequent states. |
| 5 | `ContentType` | NVARCHAR(100) | NOT NULL | — | — | MIME type of the uploaded file. Used by the API to set the correct `Content-Type` header when serving the file for download. Expected values: `application/pdf`, `application/vnd.openxmlformats-officedocument.wordprocessingml.document` (DOCX), `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet` (XLSX), `image/png`, `image/jpeg`. |
| 6 | `FileSize` | BIGINT | NOT NULL | — | — | File size in bytes. Used for validation (max 10MB = 10,485,760 bytes per BR-8.2.14) and for display purposes. The 10MB limit is enforced in the C# service layer, not via a CHECK constraint — this keeps the limit configurable without schema changes. |
| 7 | `FileData` | VARBINARY(MAX) | NOT NULL | — | — | The actual file content stored as binary data. VARBINARY(MAX) supports up to 2GB — well above the 10MB application limit. |
| 8 | `UploadedById` | INT | NOT NULL | — | FK_FileAttachments_Users_UploadedById | The user who uploaded this file. Per the Security Matrix, only the CC Owner can upload files: Supporting Documents in Initiated state, Implementation Evidence in In Implementation state. |
| 9 | `UploadedOn` | DATETIME2(7) | NOT NULL | SYSUTCDATETIME() | DF_FileAttachments_UploadedOn | Timestamp of when the file was uploaded. UTC time. |

**CHECK constraint — CK_FileAttachments_FieldName:**

```sql
CHECK (FieldName IN ('SupportingDocuments', 'ImplementationEvidence'))
```

**UNIQUE constraint — UQ_FileAttachments_ChangeControlId_FieldName:**

```sql
UNIQUE (ChangeControlId, FieldName)
```

This composite UNIQUE constraint enforces the single-file-per-field rule (BR-8.2.15). For any given CC record, there can be at most one row with `FieldName = 'SupportingDocuments'` and at most one row with `FieldName = 'ImplementationEvidence'`. This means a CC record can have 0, 1, or 2 rows in this table.

**File replacement strategy:**

When a user uploads a new file to a field that already has a file, the C# service layer handles this within a transaction:

1. Delete the existing FileAttachments row for that `(ChangeControlId, FieldName)` combination
2. Insert a new row with the new file data
3. Commit the transaction

This is simpler than an UPDATE because it cleanly replaces all columns (FileName, ContentType, FileSize, FileData, UploadedById, UploadedOn) in one operation. The UNIQUE constraint is satisfied because the old row is deleted before the new row is inserted within the same transaction.

**File type validation:**

The BRD specifies 5 allowed file types (BR-8.2.13): PDF, DOCX, XLSX, PNG, JPG. This is validated in the C# service layer by checking the `ContentType` and/or file extension — not via a CHECK constraint on `ContentType`. Rationale: MIME types can vary (e.g., JPEG files may have `image/jpeg` or `image/jpg`), and validating by extension in addition to MIME type is a service-layer concern.

**Why all columns are NOT NULL:**

Unlike the ChangeControls table where user-editable fields are nullable for Save Draft, every column in FileAttachments is NOT NULL. This is because a file attachment row is only created when an actual file is uploaded — there is no "draft" state for a file. The row either exists (file uploaded) or doesn't exist (no file yet). A row with NULL FileData or NULL FileName would be meaningless.

---

## 3.4 AuditLogs Table

**Purpose:** Append-only audit trail capturing all significant business actions within the Change Control module. Audit records are never deleted, never modified, and never overwritten (BR-8.7.7). The table structure matches the specification in BRD Section 6.6.3.

**Total columns:** 11
**Primary key:** `Id` (BIGINT IDENTITY)
**Foreign keys:** 1 (PerformedById → Users.Id)
**CHECK constraints:** 2 (EntityType, ActionType) — per approved review changes
**BRD references:** Section 6.6 (Audit Trail & History), Section 8.7 (Audit & Data Retention)

| # | Column | Data Type | Nullable | Default | Constraints | BRD Mapping |
|---|--------|-----------|----------|---------|-------------|-------------|
| 1 | `Id` | BIGINT IDENTITY(1,1) | NOT NULL | Auto-increment | PK_AuditLogs | Audit ID — unique identifier for each audit entry. |
| 2 | `EntityType` | NVARCHAR(50) | NOT NULL | — | CK_AuditLogs_EntityType | The type of entity being audited. Closed set for Phase 1. |
| 3 | `EntityId` | NVARCHAR(50) | NOT NULL | — | — | The business identifier of the specific entity (e.g., "CC-001", "User-123"). Stored as a string, not an FK — see Section 2.2.4 for rationale. |
| 4 | `ActionType` | NVARCHAR(50) | NOT NULL | — | CK_AuditLogs_ActionType | The category of action performed. 7 valid values for Phase 1. |
| 5 | `ActionDescription` | NVARCHAR(500) | NOT NULL | — | — | Human-readable summary of the action, generated by the C# AuditService at runtime. Examples: "Change Control CC-001 created", "State changed from Initiated to Pending Implementation Approval", "Decision set to Approve", "User Bob Johnson added with role CC Owner". |
| 6 | `FieldName` | NVARCHAR(100) | NULL | — | — | The name of the field that was changed. NULL for non-field actions such as record creation or state transitions where the field is implicit (CurrentState). Populated for field-level changes (e.g., "Decision", "Risk Level", "Target Closure Date"). |
| 7 | `OldValue` | NVARCHAR(MAX) | NULL | — | — | The previous value of the field before the change. NULL for creation events (no previous value exists). For field updates, captures the value that is being replaced. NVARCHAR(MAX) accommodates full field values up to 2000 characters. |
| 8 | `NewValue` | NVARCHAR(MAX) | NULL | — | — | The new value of the field after the change. NULL is rare but possible (e.g., if a field were cleared, though current business rules don't allow clearing a field once set). For creation events, may contain the initial value (e.g., "CC-001" for the CC-ID). |
| 9 | `PerformedById` | INT | NOT NULL | — | FK_AuditLogs_Users_PerformedById | The user who performed the audited action. FK to Users.Id. Every audit entry must have a performing user — there are no system-initiated actions without a user context in Phase 1. |
| 10 | `PerformedByName` | NVARCHAR(200) | NOT NULL | — | — | **Denormalized snapshot** of the performing user's full name at the time of the action. This is deliberately redundant with the FK. Rationale: audit records are immutable (BR-8.7.7). If an Admin later changes a user's FullName, the audit entry must still show the name as it was when the action occurred, not the current name. Without this denormalized column, a JOIN to Users would return the updated name, misrepresenting the historical record. |
| 11 | `Timestamp` | DATETIME2(7) | NOT NULL | SYSUTCDATETIME() | DF_AuditLogs_Timestamp | When the action occurred. UTC time. Multiple audit entries from the same action (e.g., 3 field updates + 1 state transition during an approval) share the same timestamp value (BR-8.7.5). The C# AuditService generates the timestamp once and passes it to all entries in the batch. |

**CHECK constraint — CK_AuditLogs_EntityType:**

```sql
CHECK (EntityType IN ('ChangeControl', 'User'))
```

This is a closed set for Phase 1. If future modules introduce new entity types (e.g., 'CAPA', 'Deviation'), the constraint is altered via `ALTER TABLE ... DROP/ADD CONSTRAINT`.

**CHECK constraint — CK_AuditLogs_ActionType:**

```sql
CHECK (ActionType IN (
    'Created',
    'StateChanged',
    'FieldUpdated',
    'UserAdded',
    'UserRoleChanged',
    'UserUpdated',
    'UserDeactivated'
))
```

**ActionType value definitions:**

| ActionType | Used When | EntityType | Example ActionDescription |
|------------|-----------|------------|--------------------------|
| `Created` | A new CC record is created | ChangeControl | "Change Control CC-001 created" |
| `StateChanged` | A CC record transitions between states | ChangeControl | "State changed from Initiated to Pending Implementation Approval" |
| `FieldUpdated` | A critical field value is changed on a CC record | ChangeControl | "Decision set to Approve" |
| `UserAdded` | Admin creates a new user account | User | "User Bob Johnson added with role CC Owner" |
| `UserRoleChanged` | Admin changes a user's role | User | "User Bob Johnson role changed" |
| `UserUpdated` | Admin edits a user's Full Name | User | "User Bob Johnson name updated" |
| `UserDeactivated` | Admin deactivates a user account | User | "User Bob Johnson deactivated" |

**Note on UserUpdated:** This ActionType was added during the BRD review (Point 1 of BRD corrections). It covers the case where an Admin edits a user's Full Name (the only editable user field besides Role). Documented in BRD Section 5, user story US-AD-05: Edit User Profile. Without this ActionType, name changes would be unaudited — which would break compliance traceability since the user's name appears in CC records (Change Owner, Last Updated By) and audit entries (PerformedByName).

**Critical fields tracked in the audit log (BR-8.7.2):**

The following fields generate `FieldUpdated` audit entries when their values change:

| Field | Column Name | When Changed |
|-------|-------------|-------------|
| Decision | Decision | Approver submits decision at Implementation Approval gate |
| Decision Comments | DecisionComments | Same as above |
| Risk Level | RiskLevel | Same as above |
| Final Decision | FinalDecision | Approver submits decision at Final Approval gate |
| Final Comments | FinalComments | Same as above |
| Cancellation Reason | CancellationReason | CC Owner cancels a CC |
| Target Closure Date | TargetClosureDate | CC Owner sets or changes this date |
| Proposed Implementation Date | ProposedImplementationDate | CC Owner changes this date |
| Assign Approver | AssignedApproverId | CC Owner selects or changes the Approver |

**Non-critical fields NOT tracked (BR-8.7.3):**

Edits to Change Description, Business Impact, Risk Rationale, Implementation Summary, and all other free-text content fields do NOT generate audit entries. The audit log captures only significant, compliance-relevant changes.

**Audit log entry granularity (BR-8.7.5):**

Each critical field change is a separate audit entry, even when multiple fields change in a single action. All entries from the same action share the same Timestamp value.

**Example — Approver submits Decision = 'Approve', Risk Level = 'Low', Decision Comments = 'Looks good':**

This single action generates 4 audit entries, all with the same timestamp:

| Entry | ActionType | FieldName | OldValue | NewValue |
|-------|-----------|-----------|----------|----------|
| 1 | FieldUpdated | Decision | NULL (or 'Reject' if re-review) | Approve |
| 2 | FieldUpdated | Risk Level | NULL (or previous value) | Low |
| 3 | FieldUpdated | Decision Comments | NULL (or previous comment) | Looks good |
| 4 | StateChanged | Current State | Pending Implementation Approval | In Implementation |

**Why BIGINT for the primary key:**

Audit logs grow indefinitely with no purge (BR-8.7.7). A single CC lifecycle (creation through closure) generates approximately 10–15 audit entries. A CC that goes through rejection cycles generates more. User management actions add further entries. With INT (max ~2.1 billion), a high-volume system could theoretically approach the limit over many years. BIGINT (max ~9.2 quintillion) eliminates this concern entirely. The additional 4 bytes per row (8 vs 4) is negligible compared to the NVARCHAR(MAX) columns.

**Why PerformedByName is NOT populated via JOIN at query time:**

This is a common question during design review. The answer: audit records are immutable. Consider this scenario:

1. On January 15, user "Jane Smith" (Users.Id = 5) approves CC-001
2. Audit entry created: PerformedById = 5, PerformedByName = "Jane Smith"
3. On March 1, Admin changes the user's FullName from "Jane Smith" to "Jane Al-Rashid"
4. If PerformedByName were resolved via JOIN, the audit entry would now show "Jane Al-Rashid" — misrepresenting who the system recorded as performing the action in January

Storing the name as a denormalized snapshot preserves the historical truth. The FK (PerformedById) still provides referential integrity and the ability to query all actions by a specific user across name changes.

---

### Section 3 — Complete Table Summary

| Table | Columns | PK Type | FK Count | CHECK Count | Purpose |
|-------|---------|---------|----------|-------------|---------|
| Users | 8 | INT IDENTITY | 0 | 1 (Role) | User accounts and RBAC |
| ChangeControls | 49 | INT IDENTITY | 5 (all → Users) | 11 (all dropdowns) | Main CC record — 50 BRD fields |
| FileAttachments | 9 | INT IDENTITY | 2 (→ CC, → Users) | 1 (FieldName) | File uploads for fields #24, #34 |
| AuditLogs | 11 | BIGINT IDENTITY | 1 (→ Users) | 2 (EntityType, ActionType) | Append-only audit trail |
| **Total** | **77** | | **8** | **15** | |

---

*End of Section 3*

---

**Section 3 Verification (all 3 parts combined):**

- [x] Users table: 8 columns fully defined with all constraints
- [x] ChangeControls table: 49 columns fully defined, grouped by BRD section (11 groups)
- [x] All 50 BRD fields mapped (48 in ChangeControls + 2 in FileAttachments)
- [x] Column count verification: 49 columns confirmed, 13 system-generated + 37 user-editable = 50 BRD fields ✓
- [x] FileAttachments table: 9 columns fully defined with UNIQUE constraint explanation
- [x] AuditLogs table: 11 columns fully defined with denormalization rationale
- [x] CHECK constraint on AuditLogs.EntityType added (Change 1 from review)
- [x] CHECK constraint on AuditLogs.ActionType with 7 values including UserUpdated (Change 2 from review)
- [x] All 15 CHECK constraints across all tables defined with exact SQL
- [x] All NULL/NOT NULL decisions documented with rationale
- [x] File replacement strategy documented
- [x] Audit entry granularity with concrete example (4 entries per approval action)
- [x] PerformedByName denormalization rationale with scenario walkthrough
