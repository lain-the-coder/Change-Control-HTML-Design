# SECTION 3 — Table Definitions (Part 2 of 3)

Continuing the ChangeControls table definition from Part 1 (which covered fields 1–28). This part covers fields 29–50 and the column count verification.

---

### 3.2.6 Implementation Details (BRD Fields 29–34) — CC Owner Editable in In Implementation State

Six fields capturing the actual outcomes of the implementation. These fields are only editable by the CC Owner in the In Implementation state. In earlier states (Initiated, Pending Implementation Approval), the UI displays "Not applicable — Available after approval." Note: Field 34 (Implementation Evidence) is a file upload stored in the FileAttachments table.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 29 | `ActualImplementationDate` | DATE | NULL | — | — | 29 | Actual Implementation Date |
| 30 | `PostImplementationIssues` | NVARCHAR(50) | NULL | — | CK_ChangeControls_PostImplementationIssues | 30 | Post-Implementation Issues |
| 31 | `ImplementationSummary` | NVARCHAR(2000) | NULL | — | — | 31 | Implementation Summary |
| 32 | `DeviationsFromPlan` | NVARCHAR(2000) | NULL | — | — | 32 | Deviations from Plan |
| 33 | `ValidationPerformed` | NVARCHAR(2000) | NULL | — | — | 33 | Validation Performed |
| — | *(Not in this table)* | — | — | — | — | 34 | Implementation Evidence → FileAttachments |

**Column notes:**

- **ActualImplementationDate:** DATE type (retrospective — the date the change was actually implemented). No minimum lead-time validation — this is a past/current date, not a future-looking date (BRD Field 29). Mandatory when submitting for final approval (BR-8.2.2).
- **PostImplementationIssues:** Dropdown with 3 options. Mandatory when submitting for final approval. NVARCHAR(50) accommodates the longest value ('Issues requiring follow-up' = 27 chars).
- **ImplementationSummary, ValidationPerformed:** Multi-line textareas, max 2000 characters each. Both mandatory when submitting for final approval (BR-8.2.2).
- **DeviationsFromPlan:** Multi-line textarea, max 2000 characters. **Optional** — the only non-mandatory field in this group (BRD Field 32).
- **Field 34 — Implementation Evidence:** File upload (mandatory when submitting for final approval). Stored in the FileAttachments table with `FieldName = 'ImplementationEvidence'`. See Section 3.3.

**CHECK constraint — CK_ChangeControls_PostImplementationIssues:**

```sql
CHECK (PostImplementationIssues IN (
    'None', 'Minor issues resolved', 'Issues requiring follow-up'
))
```

---

### 3.2.7 Approvals — Initiation (BRD Fields 35–36) — CC Owner Editable in Initiated State

Two fields for the CC Owner's Approver selection and any comments for the Approver's consideration during review.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 34 | `AssignedApproverId` | INT | NULL | — | FK_ChangeControls_Users_AssignedApproverId | 35 | Assign Approver |
| 35 | `CommentsForApprover` | NVARCHAR(2000) | NULL | — | — | 36 | Comments for Approver |

**Column notes:**

- **AssignedApproverId:** FK to Users.Id. NULL during draft state before the CC Owner makes a selection. The dropdown in the UI is dynamically populated with only users who currently hold the Approver role (BR-8.3.2) — this filtering happens in the C# service layer, not via a DB constraint. Mandatory at submission (BR-8.2.1). Once assigned, the same Approver handles both approval gates (BR-8.3.3). Changes to this field are tracked in the audit log (BR-8.7.2).
- **CommentsForApprover:** Optional textarea, max 2000 characters. Not tracked in the audit log (non-critical field).

---

### 3.2.8 Approvals — Implementation Approval (BRD Fields 37–41)

Five fields for the Approver's decision at the first approval gate. Three are Approver-editable in the Pending Implementation Approval state (Decision, Risk Level, Decision Comments). Two are system-generated on approval (Implementation Approval By, Implementation Approval On).

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 36 | `Decision` | NVARCHAR(10) | NULL | — | CK_ChangeControls_Decision | 37 | Decision |
| 37 | `RiskLevel` | NVARCHAR(10) | NULL | — | CK_ChangeControls_RiskLevel | 38 | Risk Level |
| 38 | `DecisionComments` | NVARCHAR(2000) | NULL | — | — | 39 | Decision Comments |
| 39 | `ImplementationApprovalById` | INT | NULL | — | FK_ChangeControls_Users_ImplementationApprovalById | 40 | Implementation Approval By |
| 40 | `ImplementationApprovalOn` | DATETIME2(7) | NULL | — | — | 41 | Implementation Approval On |

**Column notes:**

- **Decision:** Dropdown with 2 options (Approve / Reject). The value of this field drives the state transition at the Implementation Approval gate: 'Approve' → In Implementation, 'Reject' → Initiated (BR-8.1.4). Mandatory when Approver submits decision (BR-8.2.3). This field is **overwritten** if the record is rejected and later re-reviewed — the old value is preserved in the audit log before overwrite (BR-8.3.7). Changes always tracked in audit log (BR-8.7.2).
- **RiskLevel:** Dropdown with 3 options. Set exclusively by the Approver, not the CC Owner (BR-8.3.5). The CC Owner provides their risk assessment in RiskRationale (field 22), but the formal Risk Level classification is an independent Approver evaluation. Mandatory when submitting decision. Overwritten on re-review, old value preserved in audit log. Changes always tracked (BR-8.7.2).
- **DecisionComments:** Textarea, max 2000 characters. Used for **both** Approve and Reject decisions — there is no separate "Rejection Comments" field (BR-8.3.8). Mandatory when submitting decision (BR-8.2.3). Overwritten on re-review, old value preserved in audit log. Changes always tracked (BR-8.7.2).
- **ImplementationApprovalById:** FK to Users.Id. System-populated with the Approver's ID when Decision = 'Approve'. **Not populated on rejection** (BR-8.3.9). NULL until an approval occurs.
- **ImplementationApprovalOn:** System-populated with the current timestamp when Decision = 'Approve'. Not populated on rejection. NULL until an approval occurs.

**CHECK constraint — CK_ChangeControls_Decision:**

```sql
CHECK (Decision IN ('Approve', 'Reject'))
```

**CHECK constraint — CK_ChangeControls_RiskLevel:**

```sql
CHECK (RiskLevel IN ('Low', 'Medium', 'High'))
```

---

### 3.2.9 Approvals — Final Approval (BRD Fields 42–45)

Four fields for the Approver's decision at the second (final) approval gate. Two are Approver-editable in the Pending Final Approval state (Final Decision, Final Comments). Two are system-generated on final approval (Final Approval By, Final Approval On).

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 41 | `FinalDecision` | NVARCHAR(10) | NULL | — | CK_ChangeControls_FinalDecision | 42 | Final Decision |
| 42 | `FinalComments` | NVARCHAR(2000) | NULL | — | — | 43 | Final Comments |
| 43 | `FinalApprovalById` | INT | NULL | — | FK_ChangeControls_Users_FinalApprovalById | 44 | Final Approval By |
| 44 | `FinalApprovalOn` | DATETIME2(7) | NULL | — | — | 45 | Final Approval On |

**Column notes:**

- **FinalDecision:** Dropdown with 2 options (Approve / Reject). Drives the state transition at the Final Approval gate: 'Approve' → Closed, 'Reject' → In Implementation (BR-8.1.4). Mandatory when Approver submits final decision (BR-8.2.4). Overwritten on re-review, old value preserved in audit log (BR-8.3.7). Changes always tracked (BR-8.7.2).
- **FinalComments:** Textarea, max 2000 characters. Used for both Approve and Reject final decisions — no separate rejection field (BR-8.3.8). Mandatory when submitting final decision (BR-8.2.4). Overwritten on re-review, old value preserved in audit log. Changes always tracked (BR-8.7.2).
- **FinalApprovalById:** FK to Users.Id. System-populated with the Approver's ID when Final Decision = 'Approve'. Not populated on rejection (BR-8.3.9). Should match `AssignedApproverId` — same Approver for both gates (BR-8.3.3), enforced in C# service layer.
- **FinalApprovalOn:** System-populated with the current timestamp when Final Decision = 'Approve'. Not populated on rejection. NULL until final approval occurs.

**CHECK constraint — CK_ChangeControls_FinalDecision:**

```sql
CHECK (FinalDecision IN ('Approve', 'Reject'))
```

---

### 3.2.10 Approvals — Status (BRD Fields 46–48) — All System-Generated

Three system-managed fields indicating the current position in the approval lifecycle. All read-only for all users in all states.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 45 | `ImplementationApprovalStatus` | NVARCHAR(20) | NOT NULL | 'Not Submitted' | CK_ChangeControls_ImplementationApprovalStatus, DF_ChangeControls_ImplementationApprovalStatus | 46 | Implementation Approval Status |
| 46 | `FinalApprovalStatus` | NVARCHAR(20) | NOT NULL | 'Not Submitted' | CK_ChangeControls_FinalApprovalStatus, DF_ChangeControls_FinalApprovalStatus | 47 | Final Approval Status |
| 47 | `ActualClosureDate` | DATETIME2(7) | NULL | — | — | 48 | Actual Closure Date |

**Column notes:**

- **ImplementationApprovalStatus:** NOT NULL with default 'Not Submitted'. System-managed — updated automatically by the C# service layer based on the current workflow state. Value mapping by state:

| Workflow State | ImplementationApprovalStatus |
|----------------|------------------------------|
| Initiated | Not Submitted |
| Pending Implementation Approval | Pending |
| In Implementation | Approved |
| Pending Final Approval | Approved |
| Closed | Approved |
| Cancelled | N/A |

- **FinalApprovalStatus:** NOT NULL with default 'Not Submitted'. Same system-managed pattern. Value mapping by state:

| Workflow State | FinalApprovalStatus |
|----------------|---------------------|
| Initiated | Not Submitted |
| Pending Implementation Approval | Not Submitted |
| In Implementation | Not Submitted |
| Pending Final Approval | Pending |
| Closed | Approved |
| Cancelled | N/A |

- **Important:** The values are "Not Submitted" — not "Not Yet Submitted". Exact string match required per BRD (BRD Section 7, Field 46 note).
- **ActualClosureDate:** DATETIME2(7). System-populated with the current timestamp when Final Decision = 'Approve' and the state transitions to Closed (FR-6.2.20). NULL in all states except Closed. Never populated for Cancelled records (BRD Field 48).

**CHECK constraint — CK_ChangeControls_ImplementationApprovalStatus:**

```sql
CHECK (ImplementationApprovalStatus IN (
    'Not Submitted', 'Pending', 'Approved', 'N/A'
))
```

**CHECK constraint — CK_ChangeControls_FinalApprovalStatus:**

```sql
CHECK (FinalApprovalStatus IN (
    'Not Submitted', 'Pending', 'Approved', 'N/A'
))
```

---

### 3.2.11 Additional Information (BRD Fields 49–50)

Two fields: a general Comments field and the Cancellation Reason field.

| # | Column | Data Type | Nullable | Default | Constraints | BRD Field # | BRD Field Name |
|---|--------|-----------|----------|---------|-------------|-------------|----------------|
| 48 | `Comments` | NVARCHAR(2000) | NULL | — | — | 49 | Comments |
| 49 | `CancellationReason` | NVARCHAR(500) | NULL | — | — | 50 | Cancellation Reason |

**Column notes:**

- **Comments:** Optional textarea, max 2000 characters. Editable by CC Owner in Initiated state only. Read-only in all other states. Always visible on the form in all states. Non-critical field — not tracked in the audit log.
- **CancellationReason:** Textarea, max **500** characters (shorter limit than other textareas per BRD Field 50 / BR-8.2.12). Mandatory when cancelling a CC (BR-8.2.5). Entered via the cancellation popup modal, not through the inline form. The field is **hidden** in all states except Cancelled — once saved, the value is permanently read-only and cannot be modified. NULL for all non-cancelled records. Changes tracked in the audit log (BR-8.7.2).

---

### 3.2.12 ChangeControls — Column Count Verification

This verification confirms that all 50 BRD fields are accounted for in the database schema.

**Columns in the ChangeControls table:**

| Group | BRD Fields | Columns | Column Names |
|-------|------------|---------|--------------|
| Surrogate PK | — | 1 | Id |
| Identification | Fields 1–6 | 6 | CcId, CurrentState, ChangeOwnerId, LastUpdatedById, CreatedOn, LastUpdatedOn |
| Change Definition | Fields 7–12 | 6 | ChangeTitle, ChangeDescription, ChangeType, ChangeCategory, DepartmentFunction, AffectedSystemsModules |
| Planning | Fields 13–16 | 4 | ProposedImplementationDate, TargetClosureDate, ImplementationWindowStart, ImplementationWindowEnd |
| Impact & Risk Assessment | Fields 17–23 | 7 | ReasonForChange, BusinessImpact, ExpectedDowntime, RequiresTesting, RequiresTraining, RiskRationale, KeyRisksMitigations |
| Field 24 (Supporting Documents) | Field 24 | 0 | → FileAttachments table |
| Impl Plan & Validation | Fields 25–28 | 4 | HighLevelImplementationPlan, ValidationApproach, SuccessCriteria, RollbackBackoutPlan |
| Implementation Details | Fields 29–33 | 5 | ActualImplementationDate, PostImplementationIssues, ImplementationSummary, DeviationsFromPlan, ValidationPerformed |
| Field 34 (Implementation Evidence) | Field 34 | 0 | → FileAttachments table |
| Approvals — Initiation | Fields 35–36 | 2 | AssignedApproverId, CommentsForApprover |
| Approvals — Impl Approval | Fields 37–41 | 5 | Decision, RiskLevel, DecisionComments, ImplementationApprovalById, ImplementationApprovalOn |
| Approvals — Final Approval | Fields 42–45 | 4 | FinalDecision, FinalComments, FinalApprovalById, FinalApprovalOn |
| Approvals — Status | Fields 46–48 | 3 | ImplementationApprovalStatus, FinalApprovalStatus, ActualClosureDate |
| Additional Information | Fields 49–50 | 2 | Comments, CancellationReason |
| **TOTAL** | | **49** | |

**BRD field accounting:**

| Category | Count | Details |
|----------|-------|---------|
| BRD fields as columns in ChangeControls | 48 | Fields 1–23, 25–33, 35–50 |
| BRD fields in FileAttachments table | 2 | Field 24 (Supporting Documents), Field 34 (Implementation Evidence) |
| **Total BRD fields represented** | **50** | **✓ Matches BRD Section 7.12 field count** |

**System-generated vs user-editable verification:**

| Category | Count | Fields |
|----------|-------|--------|
| System-generated (always read-only) | 13 | Fields 1–6 (Identification), 40–41 (Impl Approval By/On), 44–45 (Final Approval By/On), 46–48 (Status labels, Actual Closure Date) |
| User-editable | 37 | Fields 7–39, 42–43, 49–50 |
| **Total** | **50** | **✓ Matches BRD Section 7.12 verification** |

**NOT NULL columns in ChangeControls (9 columns):**

| Column | Rationale |
|--------|-----------|
| Id | IDENTITY PK — always populated |
| CcId | Generated by usp_GenerateCCID at creation |
| CurrentState | Defaults to 'Initiated' |
| ChangeOwnerId | Set to creator's ID at creation |
| LastUpdatedById | Set to creator's ID at creation |
| CreatedOn | Defaults to SYSUTCDATETIME() |
| LastUpdatedOn | Defaults to SYSUTCDATETIME() |
| ImplementationApprovalStatus | Defaults to 'Not Submitted' |
| FinalApprovalStatus | Defaults to 'Not Submitted' |

**NULL columns in ChangeControls (40 columns):**

All remaining 40 columns are nullable — 38 user-editable fields (excluding the 2 file fields in FileAttachments) plus 2 system-generated fields that are only populated on specific events (ImplementationApprovalById/On would be 4 nullable system columns: ImplementationApprovalById, ImplementationApprovalOn, FinalApprovalById, FinalApprovalOn, ActualClosureDate = 5 nullable system columns + 35 nullable user-editable columns = 40 total nullable).

---

*End of Section 3 — Part 2 of 3*

**Part 2 covers:** ChangeControls fields 29–50 (21 columns noting field 34 goes to FileAttachments) + full column count verification.

**Coming in Part 3:** 3.3 FileAttachments table (9 columns) + 3.4 AuditLogs table (11 columns).
