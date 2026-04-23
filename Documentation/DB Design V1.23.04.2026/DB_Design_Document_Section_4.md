# SECTION 4 — Relationships & Foreign Keys

This section defines every foreign key constraint in the database with its exact constraint name, column mapping, referential action, and implementation rationale. Section 2 provided the conceptual relationship overview — this section provides the implementation specification used directly in the SQL creation scripts.

**Total foreign keys:** 8
**Referential action:** NO ACTION on all (no cascading deletes or updates)
**All FKs reference surrogate INT primary keys** (`Users.Id` or `ChangeControls.Id`)

---

## 4.1 ChangeControls → Users (5 Foreign Keys)

The ChangeControls table has 5 FK columns referencing Users.Id. These represent the different user roles in a CC record's lifecycle: creator, last modifier, assigned Approver, implementation approver, and final approver.

### FK-1: ChangeOwnerId

| Property | Value |
|----------|-------|
| Constraint Name | `FK_ChangeControls_Users_ChangeOwnerId` |
| Child Table.Column | `ChangeControls.ChangeOwnerId` |
| Parent Table.Column | `Users.Id` |
| Nullable | NOT NULL |
| On Delete | NO ACTION |
| On Update | NO ACTION |
| BRD Field | Field 3 — Change Owner |

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT FK_ChangeControls_Users_ChangeOwnerId
    FOREIGN KEY (ChangeOwnerId) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** Every CC record must have a Change Owner. Set to the creating user's `Users.Id` at record creation. Immutable — never changes after creation. The C# service layer ensures only users with Role = 'CC Owner' can create CC records, but the FK itself does not enforce role — it only ensures the referenced user exists.

---

### FK-2: LastUpdatedById

| Property | Value |
|----------|-------|
| Constraint Name | `FK_ChangeControls_Users_LastUpdatedById` |
| Child Table.Column | `ChangeControls.LastUpdatedById` |
| Parent Table.Column | `Users.Id` |
| Nullable | NOT NULL |
| On Delete | NO ACTION |
| On Update | NO ACTION |
| BRD Field | Field 4 — Last Updated By |

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT FK_ChangeControls_Users_LastUpdatedById
    FOREIGN KEY (LastUpdatedById) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** Set to the creating user's ID at creation (same as ChangeOwnerId initially). Updated to the acting user's ID on every save, submission, or workflow action — regardless of that user's role. For example, when an Approver submits a decision, LastUpdatedById changes to the Approver's ID.

---

### FK-3: AssignedApproverId

| Property | Value |
|----------|-------|
| Constraint Name | `FK_ChangeControls_Users_AssignedApproverId` |
| Child Table.Column | `ChangeControls.AssignedApproverId` |
| Parent Table.Column | `Users.Id` |
| Nullable | **NULL** |
| On Delete | NO ACTION |
| On Update | NO ACTION |
| BRD Field | Field 35 — Assign Approver |

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT FK_ChangeControls_Users_AssignedApproverId
    FOREIGN KEY (AssignedApproverId) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** NULL during draft state before the CC Owner selects an Approver. Populated when the CC Owner chooses from the Approver dropdown (which only shows users with Role = 'Approver' — filtered in C#). Mandatory at submission time (validated in C# service layer, not by NOT NULL constraint). Once assigned, the same Approver handles both approval gates (BR-8.3.3). Changes to this field are tracked in the audit log (BR-8.7.2).

**Why nullable:** The Save Draft requirement (FR-6.1.12) allows the CC Owner to save the form without selecting an Approver. The FK must accept NULL to support this.

---

### FK-4: ImplementationApprovalById

| Property | Value |
|----------|-------|
| Constraint Name | `FK_ChangeControls_Users_ImplementationApprovalById` |
| Child Table.Column | `ChangeControls.ImplementationApprovalById` |
| Parent Table.Column | `Users.Id` |
| Nullable | **NULL** |
| On Delete | NO ACTION |
| On Update | NO ACTION |
| BRD Field | Field 40 — Implementation Approval By |

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT FK_ChangeControls_Users_ImplementationApprovalById
    FOREIGN KEY (ImplementationApprovalById) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** NULL until the Approver submits Decision = 'Approve' at the Implementation Approval gate. System-populated — never set directly by a user. Not populated on rejection (BR-8.3.9). In practice, this value always matches `AssignedApproverId` when populated, since only the assigned Approver can submit a decision (BR-8.3.4) — but the DB does not enforce this match; the C# service layer does.

---

### FK-5: FinalApprovalById

| Property | Value |
|----------|-------|
| Constraint Name | `FK_ChangeControls_Users_FinalApprovalById` |
| Child Table.Column | `ChangeControls.FinalApprovalById` |
| Parent Table.Column | `Users.Id` |
| Nullable | **NULL** |
| On Delete | NO ACTION |
| On Update | NO ACTION |
| BRD Field | Field 44 — Final Approval By |

```sql
ALTER TABLE ChangeControls
ADD CONSTRAINT FK_ChangeControls_Users_FinalApprovalById
    FOREIGN KEY (FinalApprovalById) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** NULL until the Approver submits Final Decision = 'Approve' at the Final Approval gate. System-populated. Not populated on rejection (BR-8.3.9). Same Approver as `AssignedApproverId` and `ImplementationApprovalById` (BR-8.3.3), enforced in C#.

---

### Summary: ChangeControls → Users FK Nullability Pattern

| FK Column | Nullable | Reason |
|-----------|----------|--------|
| ChangeOwnerId | NOT NULL | Always known — the user who created the record |
| LastUpdatedById | NOT NULL | Always known — set at creation, updated on every action |
| AssignedApproverId | NULL | Not yet selected during draft state |
| ImplementationApprovalById | NULL | Only populated on implementation approval event |
| FinalApprovalById | NULL | Only populated on final approval event |

The pattern is straightforward: FKs that are set at creation are NOT NULL. FKs that are populated by a later event are NULL until that event occurs.

---

## 4.2 FileAttachments → ChangeControls (1 Foreign Key)

### FK-6: ChangeControlId

| Property | Value |
|----------|-------|
| Constraint Name | `FK_FileAttachments_ChangeControls_ChangeControlId` |
| Child Table.Column | `FileAttachments.ChangeControlId` |
| Parent Table.Column | `ChangeControls.Id` |
| Nullable | NOT NULL |
| On Delete | NO ACTION |
| On Update | NO ACTION |
| BRD Fields | Fields 24, 34 — Supporting Documents, Implementation Evidence |

```sql
ALTER TABLE FileAttachments
ADD CONSTRAINT FK_FileAttachments_ChangeControls_ChangeControlId
    FOREIGN KEY (ChangeControlId) REFERENCES ChangeControls (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** Every file attachment must belong to a CC record. A file cannot exist without a parent CC. Combined with the UNIQUE constraint `UQ_FileAttachments_ChangeControlId_FieldName` on `(ChangeControlId, FieldName)`, this enforces at most 2 file rows per CC record (one per upload field).

**Why NOT CASCADE DELETE:** CC records are never deleted (BR-8.7.8). If they were, we would still not cascade-delete files — file retention follows the same indefinite retention policy. NO ACTION ensures a CC record cannot be deleted if files reference it (though this scenario should never arise given the business rules).

---

## 4.3 FileAttachments → Users (1 Foreign Key)

### FK-7: UploadedById

| Property | Value |
|----------|-------|
| Constraint Name | `FK_FileAttachments_Users_UploadedById` |
| Child Table.Column | `FileAttachments.UploadedById` |
| Parent Table.Column | `Users.Id` |
| Nullable | NOT NULL |
| On Delete | NO ACTION |
| On Update | NO ACTION |

```sql
ALTER TABLE FileAttachments
ADD CONSTRAINT FK_FileAttachments_Users_UploadedById
    FOREIGN KEY (UploadedById) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** Tracks who uploaded the file. Always the CC Owner of the parent record — only CC Owners have upload permissions per the Security Matrix (Supporting Documents in Initiated state, Implementation Evidence in In Implementation state). NOT NULL because a file row is only created when an actual upload occurs.

---

## 4.4 AuditLogs → Users (1 Foreign Key)

### FK-8: PerformedById

| Property | Value |
|----------|-------|
| Constraint Name | `FK_AuditLogs_Users_PerformedById` |
| Child Table.Column | `AuditLogs.PerformedById` |
| Parent Table.Column | `Users.Id` |
| Nullable | NOT NULL |
| On Delete | NO ACTION |
| On Update | NO ACTION |

```sql
ALTER TABLE AuditLogs
ADD CONSTRAINT FK_AuditLogs_Users_PerformedById
    FOREIGN KEY (PerformedById) REFERENCES Users (Id)
    ON DELETE NO ACTION ON UPDATE NO ACTION;
```

**Behaviour:** Every audit entry must identify who performed the action. There are no system-initiated actions without a user context in Phase 1 — every auditable event is triggered by a user action (CC Owner creating/submitting, Approver deciding, Admin managing users).

**Why AuditLogs has only 1 FK (not 2 or 3):**

The audit table does NOT have FKs for the entity being audited (`EntityId`). The reasons are detailed in Section 2.2.4, but summarised here for the FK specification:

- `EntityId` stores business identifiers as strings ('CC-001', 'User-123'), not surrogate INT keys
- The audit table tracks two entity types (ChangeControl, User) — a single FK cannot reference two parent tables
- Audit records must remain self-contained and meaningful regardless of schema changes in audited tables
- The `PerformedById` FK is the only FK because "who performed the action" is always a single-type, direct, always-valid relationship

---

## 4.5 On Delete Behaviour

**All 8 foreign keys use `ON DELETE NO ACTION`.** No cascading deletes exist anywhere in the schema.

### Rationale

The NO ACTION policy is driven by three BRD requirements that collectively prohibit any physical deletion of data:

| BRD Rule | Requirement | Effect |
|----------|-------------|--------|
| BR-8.7.7 | Audit records retained indefinitely, never deleted, never modified | AuditLogs rows are permanent — nothing can trigger their deletion |
| BR-8.7.8 | CC records retained indefinitely, never deleted (including cancelled) | ChangeControls rows are permanent — nothing can trigger their deletion |
| BR-8.7.9 | User records retained even after deactivation | Users rows are permanent — deactivation sets IsActive = 0, never deletes |

**What happens if a DELETE is attempted:**

Since no record in any table is ever physically deleted in the application, the NO ACTION constraint serves as a safety net against accidental or unauthorised deletes at the database level:

- Attempting to `DELETE FROM Users WHERE Id = X` will fail if that user is referenced by any ChangeControls row (as ChangeOwnerId, LastUpdatedById, AssignedApproverId, ImplementationApprovalById, or FinalApprovalById), any FileAttachments row (as UploadedById), or any AuditLogs row (as PerformedById).
- Attempting to `DELETE FROM ChangeControls WHERE Id = X` will fail if that CC has any FileAttachments rows referencing it.
- `DELETE FROM AuditLogs` would succeed (AuditLogs has no child FKs), but the application never issues this command, and database-level permissions should restrict direct table manipulation in production.

**ON UPDATE NO ACTION:**

All FKs also specify `ON UPDATE NO ACTION`. Since all referenced columns are IDENTITY-generated surrogate keys that never change value, the ON UPDATE behaviour is technically irrelevant — but explicitly specifying NO ACTION documents the intent and prevents accidental cascading if a key were ever manually modified.

**EF Core mapping note:**

EF Core's default behaviour for required (NOT NULL) relationships is `ON DELETE CASCADE`. When scaffolding from this database, the generated `OnModelCreating` will reflect the actual NO ACTION behaviour from the database constraints. However, developers should verify after scaffolding that EF Core's `DeleteBehavior` is set to `Restrict` (which maps to NO ACTION) for all relationships, not `Cascade`. The DB-first approach means the database constraints are authoritative — EF Core must match them.

---

*End of Section 4*

---

**Section 4 Verification:**

- [x] All 8 FK constraints defined with exact constraint names following naming convention
- [x] Full SQL ALTER TABLE syntax provided for each FK
- [x] Each FK documented with: constraint name, child/parent columns, nullability, referential actions, BRD field mapping
- [x] Nullability pattern explained (creation-time FKs = NOT NULL, event-time FKs = NULL)
- [x] ChangeControls → Users: 5 FKs with individual behaviour descriptions
- [x] FileAttachments → ChangeControls: 1 FK with UNIQUE constraint cross-reference
- [x] FileAttachments → Users: 1 FK with upload permission context
- [x] AuditLogs → Users: 1 FK with rationale for why it's the only audit FK
- [x] ON DELETE NO ACTION rationale with all 3 BRD retention rules cited
- [x] ON UPDATE NO ACTION documented with IDENTITY key justification
- [x] EF Core scaffolding note for DeleteBehavior verification
- [x] Safety net behaviour described (what happens on attempted DELETE)
