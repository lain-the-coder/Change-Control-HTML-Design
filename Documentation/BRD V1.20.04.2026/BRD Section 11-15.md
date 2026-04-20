# Section 11 - 15 - Reviewed

---

## 11. INTEGRATION REQUIREMENTS

### 11.1 Email / Notification System

**IR-11.1.1:** The system shall integrate with an email delivery service to send notifications at each state transition as defined in Section 8.6.

**IR-11.1.2:** Email templates shall be maintained for each of the 7 notification types (see Section 6.4.2, Notifications N1–N7).

**IR-11.1.3:** Each email shall include the CC-ID and a summary of the required action or status update. Emails shall not contain direct clickable links to the CC record within the application.

**IR-11.1.4:** Email delivery failures shall not block the workflow action that triggered the notification. If an email fails to send, the state transition should still complete, and the failure should be logged for technical investigation.

### 11.2 File Storage

**IR-11.2.1:** The system shall provide file storage capabilities for two file upload fields: Supporting Documents (field #24) and Implementation Evidence (field #34).

**IR-11.2.2:** Supported file types: PDF, DOCX, XLSX, PNG, JPG.

**IR-11.2.3:** Maximum file size: 10MB per file.

**IR-11.2.4:** Each file upload field supports a single file upload. Users should combine related documents into one file before uploading.

**IR-11.2.5:** Uploaded files shall be associated with the specific CC record and accessible for viewing/download by any authenticated user who can view the record.

**IR-11.2.6:** Uploaded files shall be retained for the lifetime of the CC record (indefinitely).

### 11.3 User Management

**IR-11.3.1:** The Change Control module shall use a standalone user database managed within the application. There is no integration with external directory services (Azure AD, LDAP, or similar) in Phase 1.

**IR-11.3.2:** Admins manage users via the Settings → User Management interface within the application.

**IR-11.3.3:** The user database supports four role types: CC Owner, Approver, Viewer, Admin.

**IR-11.3.4:** User dropdown fields (e.g., Assign Approver) shall be populated dynamically from the internal user database, filtered by role as appropriate.

### 11.4 Future QMS Module Integrations

The following integrations are out of scope for Phase 1 but are planned for future development as additional QMS modules are built:

- **CAPA Module:** Link Change Controls to Corrective and Preventive Actions.
- **Deviation Module:** Link Change Controls to Deviation records.
- **Risk Register:** Link Change Controls to Risk Register entries.
- **Document Management System:** Integrate with a centralised document management system for controlled documents referenced by Change Controls.

These integrations will require a cross-module traceability framework that is not part of the Phase 1 scope. See Section 13.2 for planned future features.

---

## 12. ACCEPTANCE CRITERIA

### 12.1 Definition of Done

The Change Control module shall be considered complete and ready for deployment when all of the following conditions are met:

1. All 50 fields are functional, correctly validated, and display the appropriate permission state (editable, read-only, not applicable, or system-managed) per the Security Matrix.
2. All 6 workflow states operate correctly with the defined state transitions.
3. Role-based permissions are enforced at both the field level (per Security Matrix) and the action level (per Section 8.4) for all 4 roles.
4. Segregation of duties is enforced — a user cannot be both CC Owner and Approver on the same record.
5. Both rejection workflows are functional (Implementation Approval → Initiated, Final Approval → In Implementation) with correct permission reset.
6. Email notifications are sent at every state transition with correct recipients and task due dates.
7. The audit trail captures all required events (state transitions, critical field changes, user management actions) with correct old/new values and timestamps.
8. Cancellation workflow is functional — only from Initiated state, only by record owner, with mandatory reason via popup modal.
9. All mandatory field validations are enforced at submission points.
10. Date validations enforce minimum lead times (≥ 2 business days for Proposed Implementation Date, ≥ 10 business days for Target Closure Date) and block past dates at submission.
11. The implemented UI matches the approved HTML prototypes in layout, field organisation, and field display patterns.
12. All navigation views (Dashboard, All Change Controls, My Change Controls, Approvals, Settings) are functional with correct role-based visibility.

### 12.2 Test Scenarios

The following minimum test scenarios shall be executed and passed before sign-off:

| # | Scenario | Expected Outcome |
| --- | --- | --- |
| TS-01 | **Happy path:** Create → Submit → Approve → Implement → Final Approve → Close | Record moves through all 6 states correctly; all system fields populated; notifications sent at each transition |
| TS-02 | **Rejection at Implementation Approval:** Submit → Reject → Revise → Resubmit → Approve | Record loops back to Initiated; CC Owner can edit all 25 fields; resubmission succeeds; old rejection values preserved in audit log |
| TS-03 | **Rejection at Final Approval:** Submit for Final → Reject → Revise implementation → Resubmit → Approve | Record loops back to In Implementation; CC Owner can edit 6 fields; old Final Decision/Comments preserved in audit log |
| TS-04 | **Cancel CC from Initiated state** | Modal appears; Cancellation Reason required; state transitions to Cancelled; record permanently read-only; Cancellation Reason visible |
| TS-05 | **Attempt to assign self as Approver** | CC Owner's name does not appear in Approver dropdown; backend rejects submission if somehow bypassed |
| TS-06 | **Attempt to edit Target Closure Date after submission** | Field is read-only in Pending Implementation Approval and all subsequent states |
| TS-07 | **Attempt to submit with missing mandatory fields** | Submission blocked; validation errors displayed identifying missing fields |
| TS-08 | **Attempt to submit with past dates** | Submission blocked; validation error displayed (e.g., "Proposed Implementation Date cannot be in the past") |
| TS-09 | **Verify status labels update correctly at each state** | Implementation Approval Status and Final Approval Status display correct values per the mapping table in Section 7.10 |
| TS-10 | **Verify email notifications sent with correct due dates** | Each transition triggers the correct notification to the correct recipient with the correct task due date |
| TS-11 | **Verify audit trail captures all events** | State transitions, critical field changes, rejection history, cancellation reason, and user management actions all logged correctly |
| TS-12 | **Viewer attempts to edit or perform actions** | All fields read-only; no action buttons displayed; no workflow actions available |
| TS-13 | **Admin attempts to edit CC or perform CC actions** | All CC fields read-only; no CC action buttons displayed; User Management functions accessible |
| TS-14 | **Non-owner CC Owner attempts to cancel another user's CC** | Cancel CC button not displayed; backend rejects action if bypassed |
| TS-15 | **Non-assigned Approver attempts to submit decision** | Submit Decision button not displayed; backend rejects action if bypassed |
| TS-16 | Attempt to change role for user with active CC records | Admin attempts to change the role of a user who owns or is assigned as Approver on an active CC. System blocks the change and displays error listing active CC-IDs. |

### 12.3 Sign-off Requirements

Final sign-off shall require:

1. **All test scenarios passed** — every scenario in Section 12.2 has been executed and verified.
2. **Security Matrix validated** — field permissions verified for all 24 state/role combinations (6 states × 4 roles) against the Security Matrix Excel.
3. **Audit trail reviewed** — a sample audit trail from a complete happy-path scenario and a rejection scenario has been reviewed and confirmed to capture all required events with correct data.
4. **UI review completed** — the implemented interface has been compared against the HTML prototypes and confirmed to match in layout, structure, and behaviour.
5. **Stakeholder approval** — the business stakeholder has reviewed the delivered system and confirmed it meets the requirements defined in this BRD.

---

## 13. KNOWN LIMITATIONS & FUTURE ENHANCEMENTS

### 13.1 Phase 1 Limitations

The following limitations are known, accepted, and documented for the Phase 1 release. These are not defects — they are intentional scope boundaries.

**L1 — No Emergency Fast-Track Workflow:**
All changes follow the same six-state approval process regardless of urgency. The "Emergency" category has been removed from the Change Category dropdown. A future enhancement may introduce a fast-track approval path for emergency changes with shorter SLAs and streamlined approval.

**L2 — No CC Owner Delegation:**
The CC Owner of a record cannot transfer ownership to another user. If a CC Owner is unavailable (e.g., leave, resignation), there is no mechanism to reassign the record to a different CC Owner. A future enhancement may add a transfer ownership capability accessible by Admin.

**L3 — No Cross-Module Traceability:**
Change Controls cannot be linked to CAPA, Deviation, or Risk Register records. Each module operates independently. A future enhancement will introduce cross-module linking and traceability as additional QMS modules are built.

**L4 — No Stale Record Auto-Escalation:**
There is no automatic detection or escalation when approvers or CC Owners miss their task due dates. The system does not send reminder emails, reassign tasks, or flag overdue records in the UI. Mitigation: Task due dates are communicated via email notifications, and the audit table can be queried to identify overdue records for manual management review. A future enhancement may add auto-escalation, reminder emails, and overdue indicators in the UI.

**L5 — No Audit Trail UI Viewer:**
The audit trail is captured in a database table but there is no user interface to view audit history within the application. Audit data must be accessed through database queries or reporting tools. A future enhancement will add an audit history tab within the CC form showing the chronological history of all changes and actions.

**L6 — No External Directory Integration:**
User management is handled through a standalone internal database. There is no integration with Azure AD, LDAP, or other external directory services. A future enhancement may add single sign-on (SSO) and directory synchronisation.

**L7 — No Direct Links in Emails:**
Email notifications include the CC-ID and action summary but do not contain direct clickable links to the CC record. Users must navigate to the application and locate the record manually. A future enhancement may add deep links.

**L8 — No Public Holiday Calendar:**
Business day calculations (for date validations and task due dates) exclude Saturdays and Sundays only. Public holidays are not accounted for. A future enhancement may introduce a configurable public holiday calendar.

### 13.2 Planned Future Features

The following features are documented for consideration in future phases:

1. Emergency/fast-track change workflow with shorter SLAs
2. CC Owner delegation and ownership transfer
3. Cross-module traceability (CAPA, Deviation, Risk Register linking)
4. Auto-escalation and reminder emails for overdue tasks
5. Audit trail viewer UI within the CC form
6. External directory integration (Azure AD / SSO)
7. Direct deep links to CC records in email notifications
8. Configurable public holiday calendar for business day calculations
9. Reporting and analytics dashboards
10. Bulk operations (batch approval, batch status updates)
11. Mobile-optimised interface
12. Digital signatures / e-signature integration
13. Multi-language support

---

## 14. ASSUMPTIONS & DEPENDENCIES

### 14.1 Assumptions

**A1:** Users will access the application through modern web browsers on desktop or laptop devices. Mobile access is not a primary use case for Phase 1.

**A2:** The organisation has an email infrastructure capable of sending transactional emails. The specific email service provider is an implementation decision.

**A3:** The number of concurrent users will be within the range typical for an internal enterprise quality management tool (tens of users, not thousands).

**A4:** Admins will maintain proper role assignments, ensuring users are assigned the single role that reflects their primary function. The system enforces segregation of duties at the record level, but clean role management is an organisational responsibility.

**A5:** All users have a basic level of computer literacy and familiarity with web-based form interfaces. No specialised training beyond standard onboarding is assumed.

**A6:** Business days are defined as Monday through Friday, excluding Saturdays and Sundays. Public holidays are not factored into business day calculations in Phase 1.

**A7:** The application will be deployed in a single-timezone context. Multi-timezone date handling is not required for Phase 1.

### 14.2 Dependencies

**D1 — Email Service:** The notification system depends on a functioning email delivery service being available and configured. Notifications cannot be sent without this dependency.

**D2 — File Storage:** The system requires persistent storage for uploaded files (up to 10MB per file) associated with CC records. Files must be retained for the lifetime of the record. The specific storage mechanism (database, file system, or other) is an implementation decision.

**D3 — Database:** The application depends on a database system capable of supporting the CC record storage, user management, and audit trail with indefinite data retention and no automatic purging.

**D4 — Hosting Environment:** The application depends on a web hosting environment capable of serving a web application over HTTPS with support for user sessions and concurrent access.

### 14.3 Constraints

**C1 — No Tech Stack Prescription:** This BRD intentionally does not prescribe a technology stack. The choice of frontend framework, backend language, database engine, and hosting platform are implementation decisions to be made during the technical design phase.

**C2 — Phase 1 Scope Only:** This BRD covers Phase 1 of the Change Control module only. Features listed in Section 13.2 are explicitly out of scope and must not be implemented unless approved through a separate change request.

**C3 — Security Matrix as Authority:** The Security Matrix Excel (`Security_Matrix_V1_0.xlsx`) is the authoritative source for field-level permissions. In the event of a discrepancy between the BRD narrative and the Security Matrix Excel, the Security Matrix takes precedence for field permission questions.

**C4 — HTML Prototypes as Visual Reference:** The HTML prototypes serve as the visual reference for UI layout and structure. The implemented interface should match these prototypes. Any deviations from the prototypes require explicit approval.

---

## 15. APPENDICES

### Appendix A: Glossary & Acronyms

| Term | Definition |
| --- | --- |
| **CC** | Change Control — a formal record documenting a proposed, in-progress, or completed change |
| **CC Owner** | The user who creates and drives a Change Control through its lifecycle |
| **Approver** | The user who reviews and approves or rejects a Change Control at the two approval gates |
| **Viewer** | A user with read-only access to all Change Controls |
| **Admin** | A user who manages system settings and user accounts |
| **QMS** | Quality Management System |
| **EAMI** | Organisation name |
| **BRD** | Business Requirements Document |
| **SLA** | Service Level Agreement |
| **RBAC** | Role-Based Access Control |
| **CAPA** | Corrective and Preventive Action (future QMS module) |
| **Segregation of Duties** | The principle that different people must handle different stages of a critical process to prevent conflicts of interest |
| **Security Matrix** | The Excel-based reference document defining field-level permissions by role and state |

### Appendix B: Complete Field List (50 fields)

Refer to **Section 7.12** for the complete field summary table listing all 50 fields with their section, type, mandatory status, and editability. The table is not duplicated here to maintain a single source of truth and avoid synchronisation issues.

### Appendix C: Gap Analysis Resolution Summary

The following gaps were identified during pre-development analysis and have been resolved. All resolutions are incorporated into the relevant BRD sections.

| Gap | Description | Resolution | BRD Section |
| --- | --- | --- | --- |
| GAP 1 | Rejection Workflow | Rejection returns to previous state; audit trail preserves old values | 3.5, 8.1 |
| GAP 2 | Emergency Category | Removed from Phase 1; documented as limitation | 13.1 (L1) |
| GAP 3a | Estimated Effort field | Replaced with Target Closure Date | 7.3, 8.2 |
| GAP 3b | Audit Table | Database-only audit trail; no UI in Phase 1 | 6.6, 8.7 |
| GAP 3c | Task Due Dates | Email notifications only; no UI calendar | 6.4, 8.5 |
| GAP 3d | Actual Closure Date | System-generated on closure | 7.10 |
| GAP 4 | Segregation of Duties | CC Owner ≠ Approver enforced at UI and backend | 2.4, 8.3 |
| GAP 5 | CC Owner Delegation | Deferred to future phase | 13.1 (L2) |
| GAP 6 | Risk Level Ownership | Approver-only field; CC Owner cannot set | 7.8, 8.3 |
| GAP 7 | Status Labels | Corrected to "Not Submitted" / "Pending" / "Approved" / "N/A" | 7.10, 9.3 |
| GAP 8 | Change Owner field | Auto-populated from creator; removed manual dropdown | 7.1, 8.1 |
| GAP 9 | Linked Records / Traceability | Deferred to future phase | 13.1 (L3) |
| GAP 12 | Date Validation Rules | ≥ 2 business days (implementation date), ≥ 10 business days (closure date) | 8.2 |

### Appendix D: HTML Prototypes Reference

The following HTML prototype files are available in the project files and serve as the visual reference for the implemented UI:

| # | File | Description |
| --- | --- | --- |
| 1 | `login.html` | Login page |
| 2 | `forgot-password.html` | Forgot password page |
| 3 | `reset-password.html` | Password reset page |
| 4 | `email-reset-password.html` | Password reset email template |
| 5 | `dashboard-cc-owner.html` | Dashboard — CC Owner view |
| 6 | `dashboard-approver.html` | Dashboard — Approver view |
| 7 | `dashboard-empty.html` | Dashboard — empty state |
| 8 | `all-change-controls.html` | All Change Controls list view |
| 9 | `my-change-controls.html` | My Change Controls list view |
| 10 | `my-change-controls-empty.html` | My Change Controls — empty state |
| 11 | `approvals.html` | Approvals queue — with pending items |
| 12 | `approvals-empty.html` | Approvals queue — empty state |
| 13 | `cc-form-initated-state.html` | CC form — Initiated state, CC Owner view (25 editable fields) |
| 14 | `cc-form-initated-state-approver-view.html` | CC form — Initiated state, Approver/Viewer view (read-only) |
| 15 | `cc-form-pending-implementation-approval-approver-view.html` | CC form — Pending Impl Approval, Approver view (3 editable) |
| 16 | `cc-form-pending-implementation-approval-user-view.html` | CC form — Pending Impl Approval, CC Owner view (read-only) |
| 17 | `cc-form-in-implementation-implementer-view.html` | CC form — In Implementation, CC Owner view (6 editable) |
| 18 | `cc-form-in-implementation-approver-view.html` | CC form — In Implementation, Approver view (read-only) |
| 19 | `cc-form-pending-final-approval-approver-view.html` | CC form — Pending Final Approval, Approver view (2 editable) |
| 20 | `cc-form-pending-final-approval-implementer-view.html` | CC form — Pending Final Approval, CC Owner view (read-only) |
| 21 | `cc-form-closed.html` | CC form — Closed state (all read-only) |
| 22 | `cc-form-cancelled.html` | CC form — Cancelled state (all read-only, Cancellation Reason visible) |
| 23 | `settings-profile.html` | Settings — Profile page |
| 24 | `settings-profile-enduser.html` | Settings — Profile page (end user view) |
| 25 | `settings-admin.html` | Settings — Admin view (User Management) |
| 26 | `global.css` | Global stylesheet |

### Appendix E: Revision History

| Version | Date | Author | Changes |
| --- | --- | --- | --- |
| 1.0 | 2026-04-14 | EAMI Project Team | Initial BRD — All gaps resolved, ready for development |