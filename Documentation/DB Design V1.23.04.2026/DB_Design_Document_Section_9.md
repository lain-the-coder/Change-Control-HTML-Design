# SECTION 9 — Security Considerations

This section documents how the database design supports the security requirements defined in BRD Section 10 (Non-Functional Requirements) and the security-relevant business rules from Section 8. Security is a shared responsibility between the database layer (structural safeguards) and the C# service layer (runtime enforcement). This section focuses on the database's contribution.

---

## 9.1 Password Storage

**Requirement:** User passwords must never be stored as plain text (BRD Section 10, NFR-10.2).

### How the Database Supports This

| Aspect | Implementation |
|--------|---------------|
| Column | `Users.PasswordHash` — NVARCHAR(500) |
| Algorithm | BCrypt with cost factor 12 |
| Storage format | BCrypt hash string (e.g., `$2a$12$...`) — 60 characters typical |
| Plain text | Never stored anywhere — not in the Users table, not in seed scripts, not in audit logs |
| Column size | NVARCHAR(500) — oversized deliberately to accommodate future algorithm changes (e.g., Argon2 produces longer hashes) without schema migration |

### What the Database Does NOT Do

The database does not perform password hashing — it stores the hash as a string. All hashing and verification happens in the C# service layer:

- **Registration/user creation:** The C# service hashes the plain-text password using BCrypt before inserting the row.
- **Login authentication:** The C# service retrieves the `PasswordHash` for the given email and uses `BCrypt.Verify(inputPassword, storedHash)` to compare. The plain-text password never reaches the database in a query.
- **Password reset (Forgot Password flow):** The C# service generates a new hash and updates the `PasswordHash` column. The old hash is overwritten — password changes are not tracked in the audit log (passwords are sensitive data that should not appear in any log).

### BCrypt Cost Factor

The cost factor (work factor) of 12 means 2^12 = 4,096 iterations of the hashing function. This provides a balance between security and performance:

| Cost Factor | Approximate Hash Time | Recommendation |
|-------------|----------------------|----------------|
| 10 | ~100ms | Minimum acceptable |
| 12 | ~300ms | **Selected — good balance for this application** |
| 14 | ~1,200ms | High security, noticeable login delay |

The cost factor is a configuration value in the C# service layer, not a database concern. If hardware improves and 12 becomes too fast (making brute-force attacks easier), the cost factor is increased in the application code. Existing hashes remain valid — BCrypt includes the cost factor in the hash string itself, so verification works across different cost factors.

### Seed Data Password Security

As documented in Section 7.1, seed data uses pre-hashed BCrypt values — never plain-text passwords in SQL scripts. The default development password is hashed offline and the hash string is placed in the INSERT statement. See Section 7.1 for the full rationale and C# hash generation example.

---

## 9.2 Role Change Restriction (BR-8.4.11)

**Requirement:** An Admin cannot change a user's role if that user is associated with any active Change Control records. A record is "active" if it is in any state other than Closed or Cancelled. The association applies when the user is the CC Owner or the Assigned Approver on the record.

### Why This Rule Exists

This rule prevents segregation of duties violations. Consider this scenario without the restriction:

1. User A (CC Owner) creates CC-001 and assigns User B (Approver)
2. Admin changes User A's role from CC Owner to Approver while CC-001 is in progress
3. User A is now an Approver — but they created CC-001
4. If User B is unavailable and CC-001 is reassigned to User A, User A would be approving their own change

The role change restriction eliminates this entire class of problems by blocking role changes while records are active.

### How the Database Supports This

The database provides three elements that enable the C# service layer to enforce this rule:

**Element 1 — Stored Procedure (`usp_CheckActiveRecordsForUser`):**

The SP (defined in Section 8.2) queries for active CC records where the user is the owner or assigned approver. It returns the list of CC-IDs preventing the change. The C# UserService calls this SP before processing any role change.

**Element 2 — Composite Indexes:**

Two composite indexes (defined in Section 5.2) optimise the active record check query:

| Index | Columns | Supports |
|-------|---------|----------|
| IX_ChangeControls_ChangeOwnerId_CurrentState | (ChangeOwnerId, CurrentState) | Finding active CCs where user is owner |
| IX_ChangeControls_AssignedApproverId_CurrentState | (AssignedApproverId, CurrentState) | Finding active CCs where user is approver |

These indexes allow SQL Server to seek directly to the user's records and filter by state without scanning the entire ChangeControls table.

**Element 3 — Single-Role Model:**

The `Users.Role` column stores a single role value with a CHECK constraint (`CK_Users_Role`). There is no `UserRoles` junction table — a user has exactly one role at any time. This structural simplicity is what makes the segregation of duties model work: a CC Owner can never appear in the Approver dropdown because the dropdown only shows users with `Role = 'Approver'`.

### Enforcement Flow

```
Admin clicks "Change Role" for a user
    → C# UserService calls usp_CheckActiveRecordsForUser(@UserId)
        → SP returns list of active CC-IDs (if any)
    → If rows returned:
        → Block the role change
        → Display error: "Cannot change role: User is associated with
           active records CC-001, CC-003. These records must be Closed
           or Cancelled before the role change can proceed."
    → If zero rows:
        → Proceed with role change
        → Update Users.Role
        → Log to AuditLogs (ActionType = 'UserRoleChanged')
```

### What the Database Does NOT Enforce

The database does not prevent the role change at the constraint level. There is no trigger or CHECK constraint that blocks an UPDATE to `Users.Role` based on the state of ChangeControls records. This is intentional — cross-table validation logic belongs in the C# service layer, not in database triggers. The SP + index combination provides the data; the service layer makes the decision.

---

## 9.3 Data Retention Rules

**Requirement:** All records in the system are retained indefinitely. No data is ever physically deleted, archived, or purged (BR-8.7.7, BR-8.7.8, BR-8.7.9).

### Retention Policy by Table

| Table | Retention Rule | BRD Reference | Enforcement Mechanism |
|-------|---------------|---------------|----------------------|
| Users | Retained even after deactivation. Deactivated users (IsActive = 0) remain in the database. | BR-8.7.9 | NO ACTION on all FKs referencing Users. Application never issues DELETE. |
| ChangeControls | Retained indefinitely, including cancelled records. No deletion, no archival. | BR-8.7.8, FR-6.5.9 | NO ACTION on all FKs. Application never issues DELETE. |
| FileAttachments | Retained as long as the parent CC record exists (indefinitely). File replacement deletes the old row and inserts a new one (within the same upload field), but this is a replace operation, not a purge. | BR-8.2.15 | NO ACTION on FK to ChangeControls. |
| AuditLogs | Retained indefinitely. Never deleted, never modified, never overwritten. Append-only. | BR-8.7.7, FR-6.6.3 | No child FKs to prevent deletion. Protection via application-level rules and DB permissions. |

### How the Database Enforces Retention

**Foreign key NO ACTION constraints:** All 8 FK relationships use `ON DELETE NO ACTION` (Section 4.5). This means:

- A User cannot be deleted if any ChangeControl, FileAttachment, or AuditLog references them
- A ChangeControl cannot be deleted if any FileAttachment references it
- In practice, since every user will eventually be referenced by at least one AuditLog entry (the 'UserAdded' entry created when the Admin creates them), no user can ever be physically deleted

**No DELETE statements in the application:** The C# service layer never issues a `DELETE` command against Users, ChangeControls, or AuditLogs. The only DELETE that occurs is the file replacement pattern in FileAttachments (delete old file row, insert new one for the same upload field), which is a controlled replace, not a purge.

**Soft-delete for users:** User "deletion" is implemented as deactivation — setting `IsActive = 0`. The user record remains with all its data intact. Deactivated users:

- Cannot log in (checked in the authentication service)
- Do not appear in the Approver dropdown (filtered by `IsActive = 1`)
- Remain referenced by their CC records and audit entries
- Can be reactivated by Admin if needed (not in Phase 1 scope, but the data model supports it)

### Production Database Permissions

To further protect against accidental data loss, the production database should be configured with restricted permissions:

| Principal | Permissions | Rationale |
|-----------|------------|-----------|
| Application service account | SELECT, INSERT, UPDATE, EXECUTE | No DELETE permission. The app never deletes, so the permission is unnecessary. Removing it prevents accidental deletes from bugs. |
| DBA / Admin account | Full (db_owner) | For maintenance, schema changes, and emergency operations only. Access should be audited and restricted. |
| Direct table access | Denied for application account | All data access goes through EF Core (for tables) and stored procedures. No ad-hoc queries from the application. |

**Note:** These are recommendations for production deployment. Development environments may have broader permissions for convenience.

---

## 9.4 Audit Log Integrity

**Requirement:** The audit log is an append-only, immutable record. Audit entries are never deleted, modified, or overwritten (BR-8.7.7, FR-6.6.3).

### How the Database Supports Audit Integrity

**Structural protection:**

| Protection | Implementation | Effect |
|-----------|----------------|--------|
| Append-only by design | The only SP that writes to AuditLogs is `usp_InsertAuditLog` — it performs INSERT only, never UPDATE or DELETE | No code path exists to modify existing entries |
| No UPDATE in application | The C# AuditService only calls the INSERT SP. There is no `UpdateAuditLog` method. | Application cannot modify audit records |
| No DELETE in application | The C# service layer never issues DELETE against AuditLogs | Application cannot remove audit records |
| BIGINT primary key | Auto-incrementing BIGINT ensures unique, sequential entry IDs that cannot be reused | Entry order is permanently recorded |
| Denormalized PerformedByName | Name captured at action time, not resolved via JOIN | Historical accuracy preserved even if user names change |
| Timestamp from application | The C# AuditService generates the timestamp and passes it to the SP, ensuring batch entries share the exact same value | Temporal integrity of related entries |

**What the database does NOT enforce:**

The database does not have a trigger or constraint that prevents UPDATE or DELETE on the AuditLogs table. SQL Server does not natively support "append-only" tables. The immutability is enforced through:

1. **Application design** — no code path exists to modify audit entries
2. **Database permissions** — the application service account should not have DELETE permission on AuditLogs in production (see Section 9.3)
3. **Operational procedures** — DBAs should not modify audit data except under documented, authorised circumstances (e.g., court order, data correction)

### Audit Coverage

The audit trail captures three categories of events (BRD Section 6.6.2):

**Category 1 — Change Control lifecycle actions:**

| Event | ActionType | Logged Fields |
|-------|-----------|---------------|
| CC created | Created | — |
| State transition | StateChanged | From state → To state |
| Critical field changed | FieldUpdated | 9 tracked fields (see Section 3.4) |

**Category 2 — User management actions (Admin):**

| Event | ActionType | Logged Fields |
|-------|-----------|---------------|
| User created | UserAdded | New user details |
| User role changed | UserRoleChanged | Old role → New role |
| User name updated | UserUpdated | Old name → New name |
| User deactivated | UserDeactivated | — |

**Category 3 — Not logged (by design):**

| Event | Rationale |
|-------|-----------|
| Non-critical field edits (Change Description, Business Impact, etc.) | Keeps audit log focused on compliance-relevant changes (BR-8.7.3) |
| Login attempts (successful or failed) | Handled by Serilog API logging, not the business audit trail (BR-8.7.10) |
| Password changes | Sensitive data — should not appear in any log |
| File upload/replacement | Not listed in BRD Section 6.6.2 as an auditable event |
| Save Draft actions | Only the resulting field values matter at submission, not intermediate saves |

### Rejection History Preservation

A key audit integrity feature: when approval fields are overwritten during a re-review cycle (after a rejection), the audit log preserves the old values before the overwrite occurs (FR-6.6.4, BR-8.7.6).

**Example scenario:**

| Step | CC Record Shows | Audit Log Contains |
|------|----------------|-------------------|
| 1. Approver rejects with Decision = 'Reject' | Decision = 'Reject' | FieldUpdated: Decision, NULL → 'Reject' |
| 2. CC Owner revises and resubmits | — | StateChanged: Initiated → Pending Implementation Approval |
| 3. Approver approves with Decision = 'Approve' | Decision = 'Approve' (overwrites 'Reject') | FieldUpdated: Decision, 'Reject' → 'Approve' |

The CC record only shows the latest value ('Approve'), but the audit log retains the complete history: the initial rejection, the resubmission, and the subsequent approval. This is critical for compliance — auditors can see every decision that was made, not just the final outcome.

---

*End of Section 9*

---

**Section 9 Verification:**

- [x] Password storage: BCrypt, cost factor 12, NVARCHAR(500), never plain text, no password in audit logs
- [x] Role change restriction: full enforcement flow documented, cross-references to SP (Section 8.2) and indexes (Section 5.2)
- [x] Segregation of duties scenario walkthrough explaining why the rule exists
- [x] Data retention: all 4 tables covered with retention rules, BRD references, and enforcement mechanisms
- [x] Production database permissions table (no DELETE for app service account)
- [x] Soft-delete pattern for users documented (IsActive = 0)
- [x] Audit log integrity: structural protections, what DB enforces vs what it doesn't
- [x] Audit coverage summary: 3 categories (CC lifecycle, user management, not logged)
- [x] Rejection history preservation with step-by-step scenario
- [x] No triggers or complex DB-level enforcement — security logic in C# service layer
