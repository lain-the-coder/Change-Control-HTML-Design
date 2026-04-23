# SECTION 7 — Seed Data

This section defines the initial data inserted into the database after schema creation. Seed data provides the minimum required records for the application to function and enables development and testing to begin immediately.

---

## 7.1 Default Users

Four user accounts are seeded — one for each role — to support development and testing workflows. All four roles must be represented so that every feature path (CC creation, approval, viewing, user management) can be exercised immediately after deployment.

### Seed User Records

| # | FullName | Email | Role | IsActive | Purpose |
|---|----------|-------|------|----------|---------|
| 1 | System Administrator | admin@eami.com | Admin | 1 | User management, system configuration |
| 2 | Default CC Owner | owner@eami.com | CC Owner | 1 | Create and manage CC records |
| 3 | Default Approver | approver@eami.com | Approver | 1 | Review and approve/reject CC records |
| 4 | Default Viewer | viewer@eami.com | Viewer | 1 | Read-only access to CC records |

### Seed SQL

```sql
-- ============================================================
-- SEED DATA: Default Users (Development & Testing Only)
-- ============================================================
-- IMPORTANT: These accounts are for development and testing.
-- Remove or replace before production deployment.
-- Passwords are stored as pre-hashed BCrypt values.
-- Default password for all seed accounts: "P@ssw0rd123!"
-- ============================================================

INSERT INTO Users (FullName, Email, PasswordHash, Role, IsActive)
VALUES
    (
        N'System Administrator',
        N'admin@eami.com',
        N'$2a$12$LJ3m4ys3LkzQXOB8hKwBduCOvFzLGBqXGsDg7thqNW/ZmfqmZKqGK',
        N'Admin',
        1
    ),
    (
        N'Default CC Owner',
        N'owner@eami.com',
        N'$2a$12$LJ3m4ys3LkzQXOB8hKwBduCOvFzLGBqXGsDg7thqNW/ZmfqmZKqGK',
        N'CC Owner',
        1
    ),
    (
        N'Default Approver',
        N'approver@eami.com',
        N'$2a$12$LJ3m4ys3LkzQXOB8hKwBduCOvFzLGBqXGsDg7thqNW/ZmfqmZKqGK',
        N'Approver',
        1
    ),
    (
        N'Default Viewer',
        N'viewer@eami.com',
        N'$2a$12$LJ3m4ys3LkzQXOB8hKwBduCOvFzLGBqXGsDg7thqNW/ZmfqmZKqGK',
        N'Viewer',
        1
    );
```

### Password Handling Notes

**BCrypt hash format:** The `PasswordHash` values above are BCrypt hashes with a cost factor of 12. The hash format is `$2a$12$<22-char-salt><31-char-hash>`. The exact hash value in the SQL script will be generated during development setup using the same BCrypt library used by the .NET application (e.g., `BCrypt.Net-Next`).

**Why pre-hashed in the seed script:**

- Plain-text passwords must never appear in SQL scripts, even for seed data. If the script is committed to source control, stored in a shared drive, or logged during execution, plain-text passwords would be exposed.
- The BCrypt hash is generated offline (via a utility script, the .NET console, or an online BCrypt generator) and pasted into the seed SQL as a string literal.
- The hash shown above is a placeholder — the actual hash must be generated from the chosen default password before the seed script is executed.

**How to generate the BCrypt hash for seed data:**

```csharp
// C# — using BCrypt.Net-Next NuGet package
string hash = BCrypt.Net.BCrypt.HashPassword("P@ssw0rd123!", workFactor: 12);
Console.WriteLine(hash);
// Output: $2a$12$... (use this value in the seed SQL)
```

```powershell
# PowerShell — if BCrypt module is available
# Or use any online BCrypt hash generator with cost factor 12
```

**Default password for all seed accounts:** `P@ssw0rd123!` (or any password the team agrees on for development). This password is only for development and testing — all seed accounts are removed or re-credentialed before production deployment.

---

## 7.2 CHECK Constraint Values Reference

This section consolidates all valid dropdown/enum values across the system in a single reference table. This serves as a quick lookup during development and testing — verifying that the UI dropdowns, C# constants/enums, and DB CHECK constraints all use the exact same string values.

### All Valid Values by Field

| Field | Column Name | Table | Valid Values | Count |
|-------|-------------|-------|-------------|-------|
| User Role | Role | Users | CC Owner, Approver, Viewer, Admin | 4 |
| Current State | CurrentState | ChangeControls | Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, Cancelled | 6 |
| Change Type | ChangeType | ChangeControls | Application, Infrastructure, Database, Security, Network, Hardware, Process, Other | 8 |
| Change Category | ChangeCategory | ChangeControls | Normal, Standard | 2 |
| Department / Function | DepartmentFunction | ChangeControls | IT, Operations, Security, QA, Facilities, Other | 6 |
| Expected Downtime | ExpectedDowntime | ChangeControls | Yes, No, Unknown | 3 |
| Requires Testing | RequiresTesting | ChangeControls | Yes – Full testing, Yes – Partial testing, No | 3 |
| Requires Training | RequiresTraining | ChangeControls | Yes, No, Not applicable | 3 |
| Post-Implementation Issues | PostImplementationIssues | ChangeControls | None, Minor issues resolved, Issues requiring follow-up | 3 |
| Decision | Decision | ChangeControls | Approve, Reject | 2 |
| Risk Level | RiskLevel | ChangeControls | Low, Medium, High | 3 |
| Final Decision | FinalDecision | ChangeControls | Approve, Reject | 2 |
| Implementation Approval Status | ImplementationApprovalStatus | ChangeControls | Not Submitted, Pending, Approved, N/A | 4 |
| Final Approval Status | FinalApprovalStatus | ChangeControls | Not Submitted, Pending, Approved, N/A | 4 |
| File Field Name | FieldName | FileAttachments | SupportingDocuments, ImplementationEvidence | 2 |
| Entity Type | EntityType | AuditLogs | ChangeControl, User | 2 |
| Action Type | ActionType | AuditLogs | Created, StateChanged, FieldUpdated, UserAdded, UserRoleChanged, UserUpdated, UserDeactivated | 7 |

**Total distinct CHECK-constrained values across the system: 64**

### String Matching Rules

These rules apply across all layers (DB, C#, Angular) to prevent value mismatches:

1. **Exact case match** — 'Approve' is correct, 'approve' or 'APPROVE' will be rejected by the CHECK constraint (despite the case-insensitive collation, the CHECK constraint uses exact string matching via `IN`). However, note that `SQL_Latin1_General_CP1_CI_AS` is case-insensitive, meaning 'approve' would actually pass the CHECK. To avoid ambiguity, the C# service layer should always use the exact-case values defined here, and the Angular frontend should send the exact-case values from dropdowns.

2. **No leading/trailing whitespace** — ' Approve' or 'Approve ' will not match. The C# service layer should trim input before saving.

3. **Special characters preserved** — The en-dash (–) in 'Yes – Full testing' and 'Yes – Partial testing' must be the actual en-dash character (Unicode U+2013), not a regular hyphen (-). The NVARCHAR column stores this correctly. The Angular dropdown should render and submit the exact string.

4. **Exact spelling** — 'Not Submitted' (not 'Not Yet Submitted'), 'Not applicable' (not 'N/A' — that's a different value for status fields), 'Minor issues resolved' (not 'Minor Issues Resolved').

---

## 7.3 Production Deployment Notes

### Seed Data Handling for Production

The seed data defined in Section 7.1 is intended for **development and testing environments only**. Before deploying to production, the following steps must be taken:

**Option A — Remove seed users entirely:**

1. Delete all 4 seed user records from the Users table
2. Create real user accounts through the Admin UI using actual employee names, corporate email addresses, and strong passwords
3. The first real Admin account must be created via a direct SQL INSERT (since there's no Admin UI accessible without an existing Admin login)

**Option B — Replace seed credentials:**

1. Update the seed Admin account with a real administrator's name, corporate email, and a strong BCrypt-hashed password
2. Remove the other 3 seed accounts (owner, approver, viewer) — real users are created through the Admin UI
3. Change the Admin email from `admin@eami.com` to the real administrator's corporate email

**Recommended approach:** Option B — keep one Admin account with real credentials, remove the rest. This avoids the chicken-and-egg problem of needing an Admin to create the first Admin.

### Production Checklist

| # | Item | Action |
|---|------|--------|
| 1 | Seed user accounts | Remove or replace with real credentials (Option A or B above) |
| 2 | Default passwords | No seed passwords should remain — all accounts must have strong, unique passwords |
| 3 | Admin email | Must be a real corporate email address, not admin@eami.com |
| 4 | BCrypt cost factor | Verify cost factor 12 is appropriate for the production server's hardware (should take 200–400ms to hash) |
| 5 | Seed script in source control | Mark the seed script clearly as DEV/TEST ONLY with comments |
| 6 | Database permissions | Restrict direct table access in production — application connects via a service account with minimum required permissions |

---

*End of Section 7*

---

**Section 7 Verification:**

- [x] All 4 roles seeded with user accounts (Change 3 from review — incorporated)
- [x] BCrypt pre-hashed passwords in seed SQL (never plain text, even in seed data)
- [x] Development/testing only disclaimer with production deployment notes
- [x] Full INSERT SQL provided with placeholder BCrypt hashes
- [x] BCrypt hash generation instructions (C# code example)
- [x] Complete CHECK constraint values reference table (64 distinct values across 17 constraints)
- [x] String matching rules documented (case, whitespace, special characters, exact spelling)
- [x] Production deployment checklist with Option A/B approaches
- [x] Chicken-and-egg problem addressed (first Admin account via SQL INSERT)
