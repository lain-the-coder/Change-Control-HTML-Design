# SECTION 5 — Indexes

This section defines all non-clustered indexes beyond the primary keys and unique constraints (which are inherently indexed). Each index is designed to support specific query patterns identified from the BRD's functional requirements — dashboard views, list pages, approval queues, and the edge case active record check.

**Index design principles:**

- **Clustered index:** Each table's primary key (`Id`) serves as the clustered index by default. This is SQL Server's standard behaviour for IDENTITY PKs and provides optimal performance for FK joins and single-record lookups.
- **Non-clustered indexes:** Created for columns frequently used in WHERE clauses, JOIN conditions, and ORDER BY clauses of anticipated queries.
- **Composite indexes:** Used where queries consistently filter on multiple columns together (e.g., owner + state for the active record check).
- **No over-indexing:** Indexes are limited to clear, justified use cases. Each index adds write overhead (maintained on INSERT/UPDATE), so only columns with demonstrated query patterns are indexed. Additional indexes can be added after observing actual query performance in production.

**Total indexes across all tables:** 13 (excluding PK clustered indexes and UNIQUE constraint indexes)

---

## 5.1 Users Indexes

The Users table has 2 non-clustered indexes supporting authentication and Approver dropdown queries.

### IX-1: UQ_Users_Email

| Property | Value |
|----------|-------|
| Index Name | `UQ_Users_Email` |
| Table | Users |
| Column(s) | `Email` |
| Type | UNIQUE NONCLUSTERED |
| Purpose | Login authentication lookups |

```sql
CREATE UNIQUE NONCLUSTERED INDEX UQ_Users_Email
    ON Users (Email);
```

**Query pattern supported:**

```sql
-- Authentication: find user by email (login)
SELECT Id, FullName, Email, PasswordHash, Role, IsActive
FROM Users
WHERE Email = @Email;
```

This is the most frequent query in the system — executed on every login attempt. The UNIQUE constraint already creates this index implicitly, but it is documented here for completeness. Case-insensitive matching is handled by the database collation (`SQL_Latin1_General_CP1_CI_AS`).

---

### IX-2: IX_Users_Role_IsActive

| Property | Value |
|----------|-------|
| Index Name | `IX_Users_Role_IsActive` |
| Table | Users |
| Column(s) | `Role`, `IsActive` |
| Type | NONCLUSTERED (composite) |
| Purpose | Approver dropdown filtering, user management lists |

```sql
CREATE NONCLUSTERED INDEX IX_Users_Role_IsActive
    ON Users (Role, IsActive);
```

**Query patterns supported:**

```sql
-- Approver dropdown: list active users with Approver role (FR-6.2.2, BR-8.3.2)
SELECT Id, FullName
FROM Users
WHERE Role = 'Approver' AND IsActive = 1;

-- Admin user management: list users by role
SELECT Id, FullName, Email, Role, IsActive
FROM Users
WHERE Role = @Role AND IsActive = @IsActive;
```

The Approver dropdown query runs whenever a CC Owner opens the Assign Approver field in the Initiated state. The composite index on `(Role, IsActive)` allows SQL Server to seek directly to 'Approver' + active users without scanning the full table.

---

## 5.2 ChangeControls Indexes

The ChangeControls table has 7 non-clustered indexes supporting dashboard views, list pages, approval queues, and the active record check for the edge case rule (BR-8.4.11).

### IX-3: UQ_ChangeControls_CcId

| Property | Value |
|----------|-------|
| Index Name | `UQ_ChangeControls_CcId` |
| Table | ChangeControls |
| Column(s) | `CcId` |
| Type | UNIQUE NONCLUSTERED |
| Purpose | Business key lookups by CC-ID |

```sql
CREATE UNIQUE NONCLUSTERED INDEX UQ_ChangeControls_CcId
    ON ChangeControls (CcId);
```

**Query patterns supported:**

```sql
-- API: get CC record by business key (e.g., GET /api/changecontrols/CC-001)
SELECT * FROM ChangeControls WHERE CcId = @CcId;

-- Audit log cross-reference: find CC by EntityId
SELECT * FROM ChangeControls WHERE CcId = @EntityId;
```

Created implicitly by the UNIQUE constraint. Documented here because it is a critical query path — the API will use `CcId` as the public-facing identifier in URLs and responses, not the surrogate `Id`.

---

### IX-4: IX_ChangeControls_CurrentState

| Property | Value |
|----------|-------|
| Index Name | `IX_ChangeControls_CurrentState` |
| Table | ChangeControls |
| Column(s) | `CurrentState` |
| Type | NONCLUSTERED |
| Purpose | Dashboard state counts, "All Change Controls" filtered views |

```sql
CREATE NONCLUSTERED INDEX IX_ChangeControls_CurrentState
    ON ChangeControls (CurrentState);
```

**Query patterns supported:**

```sql
-- Dashboard: count records per state
SELECT CurrentState, COUNT(*) AS RecordCount
FROM ChangeControls
GROUP BY CurrentState;

-- All Change Controls list: filter by state
SELECT CcId, ChangeTitle, CurrentState, CreatedOn
FROM ChangeControls
WHERE CurrentState = @State
ORDER BY CreatedOn DESC;
```

The dashboard is the application's landing page — every user sees it on login. The state count query runs on every dashboard load. With 6 possible state values, the index provides efficient grouping without a full table scan.

---

### IX-5: IX_ChangeControls_ChangeOwnerId

| Property | Value |
|----------|-------|
| Index Name | `IX_ChangeControls_ChangeOwnerId` |
| Table | ChangeControls |
| Column(s) | `ChangeOwnerId` |
| Type | NONCLUSTERED |
| Purpose | "My Change Controls" list page |

```sql
CREATE NONCLUSTERED INDEX IX_ChangeControls_ChangeOwnerId
    ON ChangeControls (ChangeOwnerId);
```

**Query pattern supported:**

```sql
-- My Change Controls: list CCs created by the logged-in CC Owner
SELECT CcId, ChangeTitle, CurrentState, CreatedOn, LastUpdatedOn
FROM ChangeControls
WHERE ChangeOwnerId = @UserId
ORDER BY CreatedOn DESC;
```

The "My Change Controls" page shows all CC records owned by the logged-in user. This is a primary navigation view for CC Owners, accessed frequently.

---

### IX-6: IX_ChangeControls_AssignedApproverId

| Property | Value |
|----------|-------|
| Index Name | `IX_ChangeControls_AssignedApproverId` |
| Table | ChangeControls |
| Column(s) | `AssignedApproverId` |
| Type | NONCLUSTERED |
| Purpose | Approver's pending approvals queue |

```sql
CREATE NONCLUSTERED INDEX IX_ChangeControls_AssignedApproverId
    ON ChangeControls (AssignedApproverId);
```

**Query pattern supported:**

```sql
-- Approvals page: list CCs assigned to the logged-in Approver
SELECT CcId, ChangeTitle, CurrentState, CreatedOn
FROM ChangeControls
WHERE AssignedApproverId = @UserId
ORDER BY CreatedOn DESC;

-- Pending approvals: CCs awaiting the Approver's decision
SELECT CcId, ChangeTitle, CurrentState
FROM ChangeControls
WHERE AssignedApproverId = @UserId
  AND CurrentState IN ('Pending Implementation Approval', 'Pending Final Approval');
```

The Approvals page is the primary view for users with the Approver role. It shows all CCs assigned to them, with emphasis on those in pending states.

---

### IX-7: IX_ChangeControls_CreatedOn

| Property | Value |
|----------|-------|
| Index Name | `IX_ChangeControls_CreatedOn` |
| Table | ChangeControls |
| Column(s) | `CreatedOn DESC` |
| Type | NONCLUSTERED |
| Purpose | "All Change Controls" default sort order |

```sql
CREATE NONCLUSTERED INDEX IX_ChangeControls_CreatedOn
    ON ChangeControls (CreatedOn DESC);
```

**Query pattern supported:**

```sql
-- All Change Controls: default listing (newest first)
SELECT CcId, ChangeTitle, CurrentState, CreatedOn, LastUpdatedOn
FROM ChangeControls
ORDER BY CreatedOn DESC;
```

The "All Change Controls" list is visible to all roles and defaults to newest-first ordering. The DESC index supports this without an additional sort operation.

---

### IX-8: IX_ChangeControls_ChangeOwnerId_CurrentState

| Property | Value |
|----------|-------|
| Index Name | `IX_ChangeControls_ChangeOwnerId_CurrentState` |
| Table | ChangeControls |
| Column(s) | `ChangeOwnerId`, `CurrentState` |
| Type | NONCLUSTERED (composite) |
| Purpose | Active record check for role change restriction (BR-8.4.11) — owner side |

```sql
CREATE NONCLUSTERED INDEX IX_ChangeControls_ChangeOwnerId_CurrentState
    ON ChangeControls (ChangeOwnerId, CurrentState);
```

**Query pattern supported:**

```sql
-- usp_CheckActiveRecordsForUser: find active CCs where user is the owner
SELECT CcId
FROM ChangeControls
WHERE ChangeOwnerId = @UserId
  AND CurrentState NOT IN ('Closed', 'Cancelled');
```

This index directly supports the `usp_CheckActiveRecordsForUser` stored procedure. When an Admin attempts to change a user's role, the system checks for active CC records where the user is either the owner or the assigned Approver. This composite index allows SQL Server to seek to the specific user and then filter by state efficiently, without scanning all CC records.

---

### IX-9: IX_ChangeControls_AssignedApproverId_CurrentState

| Property | Value |
|----------|-------|
| Index Name | `IX_ChangeControls_AssignedApproverId_CurrentState` |
| Table | ChangeControls |
| Column(s) | `AssignedApproverId`, `CurrentState` |
| Type | NONCLUSTERED (composite) |
| Purpose | Active record check for role change restriction (BR-8.4.11) — approver side |

```sql
CREATE NONCLUSTERED INDEX IX_ChangeControls_AssignedApproverId_CurrentState
    ON ChangeControls (AssignedApproverId, CurrentState);
```

**Query pattern supported:**

```sql
-- usp_CheckActiveRecordsForUser: find active CCs where user is the assigned Approver
SELECT CcId
FROM ChangeControls
WHERE AssignedApproverId = @UserId
  AND CurrentState NOT IN ('Closed', 'Cancelled');
```

The companion index to IX-8. Together, these two composite indexes ensure the active record check (BR-8.4.11) runs efficiently regardless of whether the user is an owner or an Approver on the active records. The stored procedure queries both conditions and combines the results.

---

## 5.3 FileAttachments Indexes

The FileAttachments table has 1 index beyond its PK (the UNIQUE composite constraint creates its own index).

### IX-10: UQ_FileAttachments_ChangeControlId_FieldName

| Property | Value |
|----------|-------|
| Index Name | `UQ_FileAttachments_ChangeControlId_FieldName` |
| Table | FileAttachments |
| Column(s) | `ChangeControlId`, `FieldName` |
| Type | UNIQUE NONCLUSTERED (composite) |
| Purpose | Enforce single-file-per-field, file lookup by CC and field |

```sql
CREATE UNIQUE NONCLUSTERED INDEX UQ_FileAttachments_ChangeControlId_FieldName
    ON FileAttachments (ChangeControlId, FieldName);
```

**Query patterns supported:**

```sql
-- Get file for a specific CC and field (e.g., load Supporting Documents for CC-001)
SELECT FileName, ContentType, FileSize, FileData
FROM FileAttachments
WHERE ChangeControlId = @ChangeControlId AND FieldName = @FieldName;

-- Check if a file exists for a field (before upload replacement)
SELECT Id FROM FileAttachments
WHERE ChangeControlId = @ChangeControlId AND FieldName = @FieldName;

-- Get all files for a CC (for the full CC form view)
SELECT FieldName, FileName, FileSize, UploadedOn
FROM FileAttachments
WHERE ChangeControlId = @ChangeControlId;
```

Created implicitly by the UNIQUE constraint. This is the primary access pattern for files — always queried by CC + field combination. The UNIQUE property also provides the single-file-per-field business rule enforcement (BR-8.2.15).

**Note:** No separate index on `ChangeControlId` alone is needed. The composite UNIQUE index `(ChangeControlId, FieldName)` serves queries that filter by `ChangeControlId` only (SQL Server can use the leftmost column of a composite index for single-column lookups).

---

## 5.4 AuditLogs Indexes

The AuditLogs table has 3 non-clustered indexes supporting audit history queries by entity, by time, and by performing user.

### IX-11: IX_AuditLogs_EntityType_EntityId

| Property | Value |
|----------|-------|
| Index Name | `IX_AuditLogs_EntityType_EntityId` |
| Table | AuditLogs |
| Column(s) | `EntityType`, `EntityId` |
| Type | NONCLUSTERED (composite) |
| Purpose | Retrieve audit history for a specific CC or user |

```sql
CREATE NONCLUSTERED INDEX IX_AuditLogs_EntityType_EntityId
    ON AuditLogs (EntityType, EntityId);
```

**Query patterns supported:**

```sql
-- Audit history for a specific CC (future Phase 2 audit viewer in UI)
SELECT ActionType, ActionDescription, FieldName, OldValue, NewValue,
       PerformedByName, Timestamp
FROM AuditLogs
WHERE EntityType = 'ChangeControl' AND EntityId = @CcId
ORDER BY Timestamp DESC;

-- Audit history for a specific user (Admin review)
SELECT ActionType, ActionDescription, FieldName, OldValue, NewValue,
       PerformedByName, Timestamp
FROM AuditLogs
WHERE EntityType = 'User' AND EntityId = @UserIdentifier
ORDER BY Timestamp DESC;
```

This is the most important audit index. While Phase 1 has no audit viewer UI (BRD Section 6.6.2 — FR-6.6.2), the audit data is queryable directly and a future phase will add an in-app audit history viewer (BRD Section 13.2). The composite index on `(EntityType, EntityId)` supports efficient lookups for both CC and User audit histories.

---

### IX-12: IX_AuditLogs_Timestamp

| Property | Value |
|----------|-------|
| Index Name | `IX_AuditLogs_Timestamp` |
| Table | AuditLogs |
| Column(s) | `Timestamp DESC` |
| Type | NONCLUSTERED |
| Purpose | Chronological audit queries, recent activity views |

```sql
CREATE NONCLUSTERED INDEX IX_AuditLogs_Timestamp
    ON AuditLogs (Timestamp DESC);
```

**Query patterns supported:**

```sql
-- Recent audit activity (admin review, debugging)
SELECT TOP 100 EntityType, EntityId, ActionType, ActionDescription,
       PerformedByName, Timestamp
FROM AuditLogs
ORDER BY Timestamp DESC;

-- Audit entries within a date range
SELECT * FROM AuditLogs
WHERE Timestamp BETWEEN @StartDate AND @EndDate
ORDER BY Timestamp DESC;
```

Supports time-based audit queries. The DESC order optimises for "most recent first" which is the natural viewing order for audit data.

---

### IX-13: IX_AuditLogs_PerformedById

| Property | Value |
|----------|-------|
| Index Name | `IX_AuditLogs_PerformedById` |
| Table | AuditLogs |
| Column(s) | `PerformedById` |
| Type | NONCLUSTERED |
| Purpose | Query all actions performed by a specific user |

```sql
CREATE NONCLUSTERED INDEX IX_AuditLogs_PerformedById
    ON AuditLogs (PerformedById);
```

**Query pattern supported:**

```sql
-- All actions by a specific user (admin review, compliance audit)
SELECT EntityType, EntityId, ActionType, ActionDescription, Timestamp
FROM AuditLogs
WHERE PerformedById = @UserId
ORDER BY Timestamp DESC;
```

Supports compliance queries such as "show me everything user X has done in the system." Useful for internal audits and user activity reviews.

---

## Index Summary

| # | Index Name | Table | Column(s) | Type | Primary Use Case |
|---|-----------|-------|-----------|------|-----------------|
| 1 | UQ_Users_Email | Users | Email | UNIQUE | Login authentication |
| 2 | IX_Users_Role_IsActive | Users | Role, IsActive | Composite | Approver dropdown |
| 3 | UQ_ChangeControls_CcId | ChangeControls | CcId | UNIQUE | API lookups by CC-ID |
| 4 | IX_ChangeControls_CurrentState | ChangeControls | CurrentState | Single | Dashboard state counts |
| 5 | IX_ChangeControls_ChangeOwnerId | ChangeControls | ChangeOwnerId | Single | My Change Controls list |
| 6 | IX_ChangeControls_AssignedApproverId | ChangeControls | AssignedApproverId | Single | Approvals queue |
| 7 | IX_ChangeControls_CreatedOn | ChangeControls | CreatedOn DESC | Single | All CCs default sort |
| 8 | IX_ChangeControls_ChangeOwnerId_CurrentState | ChangeControls | ChangeOwnerId, CurrentState | Composite | BR-8.4.11 owner check |
| 9 | IX_ChangeControls_AssignedApproverId_CurrentState | ChangeControls | AssignedApproverId, CurrentState | Composite | BR-8.4.11 approver check |
| 10 | UQ_FileAttachments_ChangeControlId_FieldName | FileAttachments | ChangeControlId, FieldName | UNIQUE composite | Single-file-per-field |
| 11 | IX_AuditLogs_EntityType_EntityId | AuditLogs | EntityType, EntityId | Composite | Audit history per entity |
| 12 | IX_AuditLogs_Timestamp | AuditLogs | Timestamp DESC | Single | Recent activity queries |
| 13 | IX_AuditLogs_PerformedById | AuditLogs | PerformedById | Single | User activity queries |

**Index count by table:**

| Table | Indexes (excl. PK) | Notes |
|-------|-------------------|-------|
| Users | 2 | 1 UNIQUE + 1 composite |
| ChangeControls | 7 | 1 UNIQUE + 2 single + 2 composite + 1 single (DESC) + 1 single |
| FileAttachments | 1 | 1 UNIQUE composite |
| AuditLogs | 3 | 1 composite + 1 single (DESC) + 1 single |
| **Total** | **13** | |

---

*End of Section 5*

---

**Section 5 Verification:**

- [x] All 13 indexes defined with exact names following naming convention
- [x] Full CREATE INDEX SQL syntax provided for each index
- [x] Each index mapped to specific query patterns with example SQL
- [x] Users: 2 indexes (email lookup, Approver dropdown filtering)
- [x] ChangeControls: 7 indexes (business key, state filter, owner list, approver queue, sort order, 2x BR-8.4.11 composite)
- [x] FileAttachments: 1 index (UNIQUE composite for single-file-per-field)
- [x] AuditLogs: 3 indexes (entity history, chronological, user activity)
- [x] UNIQUE indexes noted as implicitly created by their constraints
- [x] Composite index leftmost-column reuse noted (FileAttachments)
- [x] DESC index direction documented where applicable
- [x] Summary table with all 13 indexes for quick reference
