# SECTION 10 — Appendix

---

## 10.1 BRD Field # → DB Column Mapping

Complete mapping of all 50 BRD fields to their database columns. This is the authoritative cross-reference between the BRD field definitions (Section 7) and the database schema.

| BRD # | BRD Field Name | DB Table | DB Column | Data Type | Nullable | System-Gen | Audit Tracked |
|-------|---------------|----------|-----------|-----------|----------|------------|---------------|
| 1 | CC-ID | ChangeControls | CcId | VARCHAR(10) | NOT NULL | Yes | — |
| 2 | Current State | ChangeControls | CurrentState | NVARCHAR(50) | NOT NULL | Yes | Yes (StateChanged) |
| 3 | Change Owner | ChangeControls | ChangeOwnerId | INT (FK → Users) | NOT NULL | Yes | — |
| 4 | Last Updated By | ChangeControls | LastUpdatedById | INT (FK → Users) | NOT NULL | Yes | — |
| 5 | Created On | ChangeControls | CreatedOn | DATETIME2(7) | NOT NULL | Yes | — |
| 6 | Last Updated On | ChangeControls | LastUpdatedOn | DATETIME2(7) | NOT NULL | Yes | — |
| 7 | Change Title | ChangeControls | ChangeTitle | NVARCHAR(200) | NULL | No | No |
| 8 | Change Description | ChangeControls | ChangeDescription | NVARCHAR(2000) | NULL | No | No |
| 9 | Change Type | ChangeControls | ChangeType | NVARCHAR(50) | NULL | No | No |
| 10 | Change Category | ChangeControls | ChangeCategory | NVARCHAR(20) | NULL | No | No |
| 11 | Department / Function | ChangeControls | DepartmentFunction | NVARCHAR(50) | NULL | No | No |
| 12 | Affected Systems / Modules | ChangeControls | AffectedSystemsModules | NVARCHAR(500) | NULL | No | No |
| 13 | Proposed Implementation Date | ChangeControls | ProposedImplementationDate | DATE | NULL | No | Yes |
| 14 | Target Closure Date | ChangeControls | TargetClosureDate | DATE | NULL | No | Yes |
| 15 | Implementation Window Start | ChangeControls | ImplementationWindowStart | TIME(0) | NULL | No | No |
| 16 | Implementation Window End | ChangeControls | ImplementationWindowEnd | TIME(0) | NULL | No | No |
| 17 | Reason for Change | ChangeControls | ReasonForChange | NVARCHAR(2000) | NULL | No | No |
| 18 | Business Impact | ChangeControls | BusinessImpact | NVARCHAR(2000) | NULL | No | No |
| 19 | Expected Downtime | ChangeControls | ExpectedDowntime | NVARCHAR(20) | NULL | No | No |
| 20 | Requires Testing | ChangeControls | RequiresTesting | NVARCHAR(50) | NULL | No | No |
| 21 | Requires Training | ChangeControls | RequiresTraining | NVARCHAR(30) | NULL | No | No |
| 22 | Risk Rationale | ChangeControls | RiskRationale | NVARCHAR(2000) | NULL | No | No |
| 23 | Key Risks & Mitigations | ChangeControls | KeyRisksMitigations | NVARCHAR(2000) | NULL | No | No |
| 24 | Supporting Documents | FileAttachments | FieldName = 'SupportingDocuments' | VARBINARY(MAX) | — | No | No |
| 25 | High-Level Implementation Plan | ChangeControls | HighLevelImplementationPlan | NVARCHAR(2000) | NULL | No | No |
| 26 | Validation Approach | ChangeControls | ValidationApproach | NVARCHAR(2000) | NULL | No | No |
| 27 | Success Criteria | ChangeControls | SuccessCriteria | NVARCHAR(2000) | NULL | No | No |
| 28 | Rollback / Backout Plan | ChangeControls | RollbackBackoutPlan | NVARCHAR(2000) | NULL | No | No |
| 29 | Actual Implementation Date | ChangeControls | ActualImplementationDate | DATE | NULL | No | No |
| 30 | Post-Implementation Issues | ChangeControls | PostImplementationIssues | NVARCHAR(50) | NULL | No | No |
| 31 | Implementation Summary | ChangeControls | ImplementationSummary | NVARCHAR(2000) | NULL | No | No |
| 32 | Deviations from Plan | ChangeControls | DeviationsFromPlan | NVARCHAR(2000) | NULL | No | No |
| 33 | Validation Performed | ChangeControls | ValidationPerformed | NVARCHAR(2000) | NULL | No | No |
| 34 | Implementation Evidence | FileAttachments | FieldName = 'ImplementationEvidence' | VARBINARY(MAX) | — | No | No |
| 35 | Assign Approver | ChangeControls | AssignedApproverId | INT (FK → Users) | NULL | No | Yes |
| 36 | Comments for Approver | ChangeControls | CommentsForApprover | NVARCHAR(2000) | NULL | No | No |
| 37 | Decision | ChangeControls | Decision | NVARCHAR(10) | NULL | No | Yes |
| 38 | Risk Level | ChangeControls | RiskLevel | NVARCHAR(10) | NULL | No | Yes |
| 39 | Decision Comments | ChangeControls | DecisionComments | NVARCHAR(2000) | NULL | No | Yes |
| 40 | Implementation Approval By | ChangeControls | ImplementationApprovalById | INT (FK → Users) | NULL | Yes | — |
| 41 | Implementation Approval On | ChangeControls | ImplementationApprovalOn | DATETIME2(7) | NULL | Yes | — |
| 42 | Final Decision | ChangeControls | FinalDecision | NVARCHAR(10) | NULL | No | Yes |
| 43 | Final Comments | ChangeControls | FinalComments | NVARCHAR(2000) | NULL | No | Yes |
| 44 | Final Approval By | ChangeControls | FinalApprovalById | INT (FK → Users) | NULL | Yes | — |
| 45 | Final Approval On | ChangeControls | FinalApprovalOn | DATETIME2(7) | NULL | Yes | — |
| 46 | Implementation Approval Status | ChangeControls | ImplementationApprovalStatus | NVARCHAR(20) | NOT NULL | Yes | — |
| 47 | Final Approval Status | ChangeControls | FinalApprovalStatus | NVARCHAR(20) | NOT NULL | Yes | — |
| 48 | Actual Closure Date | ChangeControls | ActualClosureDate | DATETIME2(7) | NULL | Yes | — |
| 49 | Comments | ChangeControls | Comments | NVARCHAR(2000) | NULL | No | No |
| 50 | Cancellation Reason | ChangeControls | CancellationReason | NVARCHAR(500) | NULL | No | Yes |

### Field Count Summary

| Category | Count | Fields |
|----------|-------|--------|
| System-generated (always read-only) | 13 | 1–6, 40–41, 44–45, 46–48 |
| User-editable | 37 | 7–39, 42–43, 49–50 |
| Stored in ChangeControls table | 48 | All except 24 and 34 |
| Stored in FileAttachments table | 2 | 24 (Supporting Documents), 34 (Implementation Evidence) |
| Audit-tracked (critical fields) | 9 | 2 (via StateChanged), 13, 14, 35, 37, 38, 39, 42, 43, 50 |
| **Total BRD fields** | **50** | **✓** |

---

## 10.2 Data Type Decisions Log

This log documents every non-obvious data type decision and the rationale behind it. Straightforward choices (e.g., NVARCHAR(2000) for a textarea with max 2000 characters) are not listed — only decisions where an alternative was considered.

| # | Column | Chosen Type | Alternative Considered | Rationale for Choice |
|---|--------|-------------|----------------------|---------------------|
| 1 | CcId | VARCHAR(10) | NVARCHAR(10) | CC-XXX format is ASCII-only. VARCHAR saves 1 byte/char. No Unicode characters possible in this format. |
| 2 | CurrentState | NVARCHAR(50) | TINYINT + lookup table | String values are self-documenting in query results and audit logs. No JOIN needed to interpret the value. CHECK constraint enforces validity. EF Core maps cleanly to C# string. The longest value ('Pending Implementation Approval' = 35 chars) fits in 50. |
| 3 | ChangeOwnerId | INT (FK) | NVARCHAR(200) storing the name directly | FK preserves referential integrity. Name resolved via JOIN. If name changes, CC automatically reflects the current name (appropriate for owner/updater display, unlike audit logs where historical name matters). |
| 4 | ProposedImplementationDate | DATE | DATETIME2(7) | BRD specifies date-only (no time component). DATE stores in 3 bytes vs 8 bytes for DATETIME2. Avoids timezone-related time comparison issues. |
| 5 | ImplementationWindowStart | TIME(0) | NVARCHAR(5) storing "02:00" | TIME(0) is a proper temporal type — supports time arithmetic if ever needed. Stores in 3 bytes. The (0) precision means no fractional seconds, matching BRD examples. |
| 6 | ExpectedDowntime | NVARCHAR(20) | BIT (Yes/No only) | BRD defines 3 values: Yes, No, Unknown. BIT can't represent 3 states cleanly (NULL as 'Unknown' is semantically wrong). NVARCHAR with CHECK constraint matches the BRD exactly. |
| 7 | RequiresTesting | NVARCHAR(50) | TINYINT (1, 2, 3) | Values contain special characters ('Yes – Full testing' has an en-dash). Integer codes would require translation in every query and UI. String values are self-documenting. |
| 8 | Decision, FinalDecision | NVARCHAR(10) | BIT (1 = Approve, 0 = Reject) | BIT can't represent NULL (not yet decided) without ambiguity. The string values 'Approve'/'Reject' are self-documenting in queries and audit logs. Consistent with the CHECK constraint pattern used across all dropdowns. |
| 9 | FileData | VARBINARY(MAX) | FILESTREAM / file system path | User confirmed DB storage (BLOB). VARBINARY(MAX) supports up to 2GB — well above the 10MB limit. Simpler deployment (no FILESTREAM configuration, no file system permissions). Backup includes all data. Trade-off: larger database size, but with max 2 files per CC and 10MB each, this is manageable. |
| 10 | FileSize | BIGINT | INT | INT (max ~2.1 billion bytes = ~2GB) would technically suffice for 10MB files. BIGINT is the conventional type for file sizes in database design — avoids any future issues if limits are raised. 4 extra bytes per row is negligible. |
| 11 | AuditLogs.Id | BIGINT IDENTITY | INT IDENTITY | Audit logs grow indefinitely (no purge). A single approval action generates 4 entries. Over years of operation with many CC records and user management actions, INT (max ~2.1 billion) could be reached. BIGINT eliminates this concern. |
| 12 | AuditLogs.OldValue / NewValue | NVARCHAR(MAX) | NVARCHAR(2000) | Most field values are ≤ 2000 characters, but NVARCHAR(MAX) provides flexibility. If field limits are increased in a future phase, audit entries don't need schema migration. MAX has no storage overhead for short strings (SQL Server stores in-row up to 8,000 bytes). |
| 13 | AuditLogs.EntityId | NVARCHAR(50) | INT (FK to ChangeControls or Users) | EntityId references two different entity types ('ChangeControl', 'User'). A single INT FK can't reference two parent tables. String identifiers ('CC-001', 'User-123') are self-contained — audit entries remain meaningful without JOINs. |
| 14 | AuditLogs.PerformedByName | NVARCHAR(200) (denormalized) | Omitted (resolve via JOIN to Users.FullName) | Audit records are immutable. If a user's name is changed by Admin after an audit entry is created, a JOIN would return the current name — misrepresenting the historical record. The denormalized snapshot preserves the name at action time. See Section 3.4 for the full walkthrough. |
| 15 | PasswordHash | NVARCHAR(500) | NVARCHAR(60) or CHAR(60) | BCrypt outputs ~60 chars. NVARCHAR(500) is deliberately oversized to accommodate future algorithm changes (Argon2, scrypt produce longer hashes) without requiring schema migration. The extra allocated-but-unused space costs nothing for NVARCHAR (variable-length). |
| 16 | Users.Role | NVARCHAR(20) | INT + lookup table | Same rationale as CurrentState (#2). 'CC Owner' is 8 chars, 'Approver' is 8 chars — all fit in 20. Self-documenting in queries. CHECK constraint enforces validity. |
| 17 | ImplementationApprovalStatus | NVARCHAR(20) with DEFAULT | Computed column based on CurrentState | A computed column would always derive the status from the current state, which seems elegant. However: (a) the mapping logic would be a CASE expression in T-SQL, duplicating the C# service layer logic; (b) computed columns interact awkwardly with EF Core scaffolding; (c) a persisted computed column adds write overhead on every UPDATE. A regular column with a DEFAULT and service-layer management is simpler and keeps logic in C#. |

---

## 10.3 Verification Checklist

This checklist is from the DB Design Handoff Document. Every item has been addressed in this document.

| # | Verification Item | Status | Where Addressed |
|---|------------------|--------|-----------------|
| 1 | All 50 BRD fields mapped to DB columns | ✅ Done | Section 3.2 (48 in ChangeControls) + Section 3.3 (2 in FileAttachments). Verified in Section 3.2.12 and Section 10.1. |
| 2 | All 13 system-generated fields have correct defaults/triggers | ✅ Done | Section 6.2 (DEFAULT constraints). Fields: CcId (SP), CurrentState ('Initiated'), ChangeOwnerId (app), LastUpdatedById (app), CreatedOn (SYSUTC), LastUpdatedOn (SYSUTC), ImplApprovalBy/On (app), FinalApprovalBy/On (app), ImplApprovalStatus ('Not Submitted'), FinalApprovalStatus ('Not Submitted'), ActualClosureDate (app). |
| 3 | All lookup values from BRD Section 7 included as seed data | ✅ Done | Section 6.1 (CHECK constraints with all valid values) + Section 7.2 (consolidated reference table with all 64 values). |
| 4 | Audit log table matches BRD Section 6.6.3 structure | ✅ Done | Section 3.4 — 11 columns matching the BRD specification exactly. |
| 5 | Foreign key relationships correct (CC ↔ Users, etc.) | ✅ Done | Section 4 — all 8 FKs defined with exact constraint names and SQL. |
| 6 | File attachment table supports single file per field, max 10MB | ✅ Done | Section 3.3 — UNIQUE constraint on (ChangeControlId, FieldName). 10MB validated in C# service layer. |
| 7 | User table supports 4 roles with single-role constraint | ✅ Done | Section 3.1 — CHECK constraint CK_Users_Role with 4 values. Single NVARCHAR column (no junction table). |
| 8 | Status label values match BRD exactly ("Not Submitted", not "Not Yet Submitted") | ✅ Done | Section 3.2.10 — exact strings documented with emphasis. Section 6.1 — CHECK constraints use exact values. |
| 9 | Edge case rule (BR-8.4.11) can be enforced via query | ✅ Done | Section 8.2 (usp_CheckActiveRecordsForUser) + Section 5.2 (two composite indexes) + Section 9.2 (enforcement flow). |
| 10 | CC-ID auto-generation mechanism defined (CC-XXX format) | ✅ Done | Section 8.3 (usp_GenerateCCID with concurrency-safe locking). |
| 11 | Password stored as hash, never plain text | ✅ Done | Section 9.1 (BCrypt, cost factor 12, NVARCHAR(500)). Section 7.1 (seed data uses pre-hashed values). |
| 12 | All NOT NULL constraints match mandatory field rules | ✅ Done | Section 1.5 (NULL strategy). Section 3.2.12 (9 NOT NULL + 40 NULL columns verified). |
| 13 | Data retention rules reflected (no cascade deletes, no auto-purge) | ✅ Done | Section 4.5 (NO ACTION on all 8 FKs) + Section 9.3 (retention policy by table). |

**All 13 verification items: ✅ PASSED**

---

## Document Summary

| Section | Content | Status |
|---------|---------|--------|
| 1 | Overview & Conventions | ✅ Complete |
| 2 | Entity Relationship Diagram | ✅ Complete |
| 3 | Table Definitions (3 parts) | ✅ Complete |
| 4 | Relationships & Foreign Keys | ✅ Complete |
| 5 | Indexes | ✅ Complete |
| 6 | Constraints & Defaults | ✅ Complete |
| 7 | Seed Data | ✅ Complete |
| 8 | Stored Procedures | ✅ Complete |
| 9 | Security Considerations | ✅ Complete |
| 10 | Appendix | ✅ Complete |

**Database schema totals:**

| Metric | Count |
|--------|-------|
| Tables | 4 |
| Total columns | 77 |
| BRD fields mapped | 50 / 50 |
| Foreign keys | 8 |
| CHECK constraints | 17 |
| DEFAULT constraints | 10 |
| UNIQUE constraints | 3 |
| Indexes (excl. PK) | 13 |
| Stored procedures | 3 |

---

*End of Section 10 — End of Database Design Document*

---

**Document Version:** 1.0
**Date:** 22 April 2026
**Status:** Complete — Ready for Review
**Next Step:** Upon approval, proceed to SQL script creation (tables, constraints, indexes, seed data, stored procedures).
