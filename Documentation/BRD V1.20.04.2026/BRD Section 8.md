# Section 8 - Reviewed

---

## 8. BUSINESS RULES

This section consolidates all business rules that govern the behaviour of the Change Control module. While many of these rules are also referenced in their respective functional requirement sections, this section serves as a centralised index that developers and testers can use to verify that every rule is implemented and validated.

Each rule is identified with a prefix (BR-) for traceability.

---

### 8.1 State Transition Rules

These rules define when and how a Change Control record moves between states.

**BR-8.1.1 — Valid State Transitions:**
The system shall only permit the following state transitions. Any transition not listed here is invalid and must be prevented:

| # | From State | To State | Trigger | Performed By |
| --- | --- | --- | --- | --- |
| T1 | *(New)* | Initiated | Create CC | CC Owner |
| T2 | Initiated | Pending Implementation Approval | Submit for Approval | CC Owner (record owner) |
| T3 | Initiated | Cancelled | Cancel CC | CC Owner (record owner) |
| T4 | Pending Implementation Approval | In Implementation | Submit Decision (Decision = Approve) | Approver (assigned) |
| T5 | Pending Implementation Approval | Initiated | Submit Decision (Decision = Reject) | Approver (assigned) |
| T6 | In Implementation | Pending Final Approval | Submit for Final Approval | CC Owner (record owner) |
| T7 | Pending Final Approval | Closed | Submit Decision (Final Decision = Approve) | Approver (assigned) |
| T8 | Pending Final Approval | In Implementation | Submit Decision (Final Decision = Reject) | Approver (assigned) |

**BR-8.1.2 — No State Skipping:**
A record cannot skip states. There is no direct path from Initiated to In Implementation, from Initiated to Closed, or from Pending Implementation Approval to Closed. Every record must pass through the full sequential workflow or be cancelled from Initiated.

**BR-8.1.3 — Terminal States:**
Closed and Cancelled are terminal states. No transitions exit from these states. Once a record reaches either state, it remains there permanently. There is no "reopen," "reactivate," or "undo" action for either terminal state.

**BR-8.1.4 — Decision Field Drives Transition:**
At both approval gates, a single "Submit Decision" button is used. The system reads the value of the Decision field (at Implementation Approval) or Final Decision field (at Final Approval) to determine the transition. There are no separate "Approve" and "Reject" buttons.

**BR-8.1.5 — Rejection Returns to Previous State:**
Rejection does not create a new state. Rejection at the Implementation Approval gate returns the record to Initiated. Rejection at the Final Approval gate returns the record to In Implementation. The record re-enters the previous state with full edit permissions for the CC Owner.

**BR-8.1.6 — System Fields Updated on Transition:**
On every state transition, the system shall automatically update:

- Current State (to the new state value)
- Last Updated By (to the user who triggered the transition)
- Last Updated On (to the current date and time)

Additionally, on specific transitions:

- T4 (Approval at Implementation gate): Implementation Approval By, Implementation Approval On, and Implementation Approval Status are populated/updated.
- T7 (Approval at Final gate): Final Approval By, Final Approval On, Final Approval Status, and Actual Closure Date are populated/updated.
- T3 (Cancellation): Implementation Approval Status and Final Approval Status are set to "N/A."

**BR-8.1.7 — Audit Log on Every Transition:**
Every state transition shall be captured in the audit log with the from-state, to-state, performing user, and timestamp. See Section 6.6 for audit trail requirements.

---

### 8.2 Field Validation Rules

These rules define the validation constraints applied to field values.

### 8.2.1 Mandatory Field Validation

**BR-8.2.1 — Mandatory Fields at Submission (Initiated → Pending Implementation Approval):**
The following fields must be populated (non-empty, non-whitespace) before the CC Owner can submit for approval: Change Title, Change Description, Change Type, Change Category, Department/Function, Affected Systems/Modules, Proposed Implementation Date, Target Closure Date, Reason for Change, Business Impact, Expected Downtime, Requires Testing, Requires Training, Risk Rationale, Key Risks & Mitigations, High-Level Implementation Plan, Validation Approach, Success Criteria, Rollback/Backout Plan, and Assign Approver.

**BR-8.2.2 — Mandatory Fields at Submission (In Implementation → Pending Final Approval):**
The following fields must be populated before the CC Owner can submit for final approval: Actual Implementation Date, Post-Implementation Issues (dropdown selection required), Implementation Summary, Validation Performed, and Implementation Evidence (file upload required).

**BR-8.2.3 — Mandatory Field at Decision (Pending Implementation Approval):**
The following fields must be populated before the Approver can submit their decision: Decision, Risk Level, and Decision Comments.

**BR-8.2.4 — Mandatory Field at Decision (Pending Final Approval):**
The following fields must be populated before the Approver can submit their final decision: Final Decision and Final Comments.

**BR-8.2.5 — Mandatory Field at Cancellation:**
The Cancellation Reason field must be populated (non-empty, non-whitespace) before the CC Owner can confirm cancellation.

**BR-8.2.6 — Validation Error Handling:**
When a mandatory field validation fails, the system shall block the action (submission, decision, or cancellation), display a clear validation error identifying the specific field(s) that need attention, and keep the user on the current form without losing any entered data.

### 8.2.2 Date Validation Rules

**BR-8.2.7 — Proposed Implementation Date Minimum Lead Time:**
The Proposed Implementation Date must be ≥ 2 business days from the current date at the time of validation. Business days exclude Saturdays and Sundays. Public holiday handling is not required in Phase 1.

**BR-8.2.8 — Target Closure Date Minimum Lead Time:**
The Target Closure Date must be ≥ 10 business days from the current date at the time of validation.

**BR-8.2.9 — Future Date Validation at Submission:**
At the time of clicking "Submit for Approval," both the Proposed Implementation Date and Target Closure Date must be in the future (greater than the current date). If either date has become past since it was originally entered (e.g., the CC Owner saved a draft days ago and is now submitting), the submission shall be blocked with a specific error message (e.g., "Proposed Implementation Date cannot be in the past. Please update.").

**BR-8.2.10 — Re-Validation After Update:**
If the CC Owner updates a date field after a validation failure, the system shall re-validate the updated date against the current date at the time of the new submission attempt. The minimum lead-time rules (≥ 2 business days for Proposed Implementation Date, ≥ 10 business days for Target Closure Date) are recalculated from the new current date.

**BR-8.2.11 — Date Validation on Resubmission After Rejection:**
When a record returns to Initiated after rejection and the CC Owner resubmits, all date validations apply fresh based on the current date at the time of resubmission. Dates that were valid at the original submission may no longer be valid if time has passed.

### 8.2.3 Character Length Validation

**BR-8.2.12 — Character Limits:**
The following maximum character limits shall be enforced:

| Field | Max Length |
| --- | --- |
| Change Title | 200 characters |
| CC-ID | 10 characters |
| Cancellation Reason | 500 characters |
| All other textarea fields (Change Description, Reason for Change, Business Impact, Risk Rationale, Key Risks & Mitigations, High-Level Implementation Plan, Validation Approach, Success Criteria, Rollback/Backout Plan, Implementation Summary, Deviations from Plan, Validation Performed, Comments for Approver, Decision Comments, Final Comments, Comments) | 2000 characters each |

### 8.2.4 File Upload Validation

**BR-8.2.13 — Supported File Types:**
File upload fields (Supporting Documents and Implementation Evidence) shall accept the following file types: PDF, DOCX, XLSX, PNG, JPG.

**BR-8.2.14 — Maximum File Size:**
Each uploaded file shall not exceed 10MB in size. If a file exceeds this limit, the upload shall be rejected with an appropriate error message.

**BR-8.2.15 — Single File Upload:**
Each file upload field (Supporting Documents and Implementation Evidence) supports a single file upload per field. Users should combine related documents into one file before uploading.

---

### 8.3 Approval Rules

These rules govern the approval process and the behaviour of approval-related fields.

**BR-8.3.1 — Segregation of Duties:**
 The CC Owner of a record and the assigned Approver must be different individuals. Since each user can hold only one role at a time, a CC Owner will never appear in the Approver dropdown (which only displays users with the Approver role). The edge case of a role change creating a conflict is prevented by BR-8.4.11 (role change restriction for active records).

**BR-8.3.2 — Approver Role Restriction:**
Only users who hold the Approver role shall appear in the "Assign Approver" dropdown. Users with CC Owner, Viewer, or Admin roles shall not be selectable as Approvers.

**BR-8.3.3 — Single Approver Per Record:**
Each Change Control record has exactly one assigned Approver. The same Approver reviews the record at both the Implementation Approval gate and the Final Approval gate.

**BR-8.3.4 — Assigned Approver Only:**
Only the Approver who is assigned to a specific record can submit a decision on that record. Other users with the Approver role cannot submit decisions on records they are not assigned to.

**BR-8.3.5 — Risk Level Is Approver-Only and Mandatory:**
 The Risk Level field is set exclusively by the Approver during the Pending Implementation Approval state and is mandatory. The CC Owner does not set Risk Level. The CC Owner provides their own risk assessment in the Risk Rationale and Key Risks & Mitigations fields, but the formal Risk Level classification is an independent, mandatory Approver evaluation.

**BR-8.3.6 — Single Decision Field, Not Separate Buttons:**
The approval/rejection mechanism uses a Decision dropdown field (Approve/Reject) combined with a single "Submit Decision" button. The system reads the field value to determine the transition. There are no separate "Approve" and "Reject" buttons anywhere in the interface.

**BR-8.3.7 — Comments Reused Across Review Cycles:**
Decision Comments and Final Comments are single fields that are overwritten on each review cycle. They are not append-only logs. When an Approver re-reviews a record after a rejection, the new comments replace the previous comments in the record. The old comments are preserved in the audit log before overwrite.

**BR-8.3.8 — No Separate Rejection Comments Field:**
There is no dedicated "Rejection Comments" field. The Decision Comments field (at the Implementation Approval gate) and the Final Comments field (at the Final Approval gate) are used for both approval and rejection rationale.

**BR-8.3.9 — Approval Timestamp Fields Populated on Approve Only:**
Implementation Approval By / On and Final Approval By / On are populated only when the decision is "Approve." They are not populated on rejection. If a record is rejected and later approved, these fields are populated at the time of the approval, not the rejection.

---

### 8.4 Action Permissions

These rules define who can perform each action and under what conditions.

**BR-8.4.1 — Create CC:**

- Who: Any user with the CC Owner role
- When: Always (the "+ Create Change Control" button is always available to CC Owners)
- Result: A new CC record is created in the Initiated state

**BR-8.4.2 — Submit for Approval:**

- Who: The CC Owner of the specific record (not any CC Owner)
- When: Record is in the Initiated state and all mandatory field validations pass
- Result: State transitions from Initiated to Pending Implementation Approval

**BR-8.4.3 — Cancel CC:**

- Who: The CC Owner of the specific record (not any CC Owner)
- When: Record is in the Initiated state ONLY. Cancellation is not available from any other state.
- Condition: Mandatory Cancellation Reason provided via popup modal
- Result: State transitions from Initiated to Cancelled (permanent)

**BR-8.4.4 — Submit Decision (Implementation Approval):**

- Who: The Approver assigned to the specific record (not any Approver)
- When: Record is in the Pending Implementation Approval state
- Condition: Decision field must be populated
- Result: State transitions to In Implementation (if Approve) or Initiated (if Reject)

**BR-8.4.5 — Submit for Final Approval:**

- Who: The CC Owner of the specific record
- When: Record is in the In Implementation state
- Result: State transitions from In Implementation to Pending Final Approval

**BR-8.4.6 — Submit Decision (Final Approval):**

- Who: The Approver assigned to the specific record (not any Approver)
- When: Record is in the Pending Final Approval state
- Condition: Final Decision field must be populated
- Result: State transitions to Closed (if Approve) or In Implementation (if Reject)

**BR-8.4.7 — View CC:**

- Who: All users with any of the four roles (CC Owner, Approver, Viewer, Admin)
- When: Always — all users can view all CC records in the system regardless of ownership or role
- Restriction: Viewing is unrestricted, but editing is governed by the Security Matrix

**BR-8.4.8 — Save Draft:**

- Who: The CC Owner of the specific record
- When: Record is in the Initiated state
- Result: Field values saved without validation; record remains in Initiated state

**BR-8.4.9 — Manage Users:**

- Who: Admin only
- When: Always (via Settings → User Management)
- Create User: Admin sets Full Name, Email, Password, and Role. The new user can sign in immediately.
- Edit User: Admin can edit Full Name and Role only. Email cannot be changed (it is the login identifier, set at creation). Password cannot be reset through the application (users use the Forgot Password flow, or passwords are managed at the database level).
- Deactivate User: Admin can deactivate a user account, preventing future login. User records are retained for audit purposes.
- Result: User changes take effect immediately; all actions logged in audit trail.
- Restriction: See BR-8.4.11 for role change restrictions.

**BR-8.4.10 — Record-Specific Ownership:**
All action permissions that reference "CC Owner of the specific record" mean the individual user who created that particular CC record. Other users who also hold the CC Owner role cannot perform owner-specific actions (Cancel, Submit for Approval, Submit for Final Approval) on records they did not create.

**BR-8.4.11 — Role Change Restriction for Active Records:**

An Admin cannot change a user's role if that user is associated with any active Change Control records. A record is considered "active" if it is in any state other than Closed or Cancelled. The association applies when the user is either the CC Owner (creator) of an active record or the assigned Approver on an active record.

When the Admin attempts a role change and active records exist:

- The role change is blocked.
- The system displays an error message listing the active CC-IDs preventing the change.
- The Admin must wait until all associated records reach a terminal state (Closed or Cancelled) before the role change can be processed.

This rule prevents segregation of duties violations that could occur if a CC Owner's role were changed to Approver (or vice versa) while they have active records. It eliminates the need for complex per-record validation at approval submission time.

---

### 8.5 Task Due Dates & SLA

These rules define how task due dates are calculated and communicated.

**BR-8.5.1 — Task 1: Implementation Approval Review:**

- Assignee: Assigned Approver
- Trigger: Record transitions to Pending Implementation Approval
- Due Date Calculation: Submission Date + 5 business days
- Communication: Email notification (Notification N1 — see Section 6.4)

**BR-8.5.2 — Task 2: Implementation Completion:**

- Assignee: CC Owner
- Trigger: Record transitions to In Implementation (via approval)
- Due Date Calculation: Target Closure Date − 3 business days
- Communication: Email notification (Notification N2 — see Section 6.4)

**BR-8.5.3 — Task 3: Final Approval Review:**

- Assignee: Assigned Approver
- Trigger: Record transitions to Pending Final Approval
- Due Date Calculation: Target Closure Date
- Communication: Email notification (Notification N4 — see Section 6.4)

**BR-8.5.4 — Business Days Definition:**
Business days exclude Saturdays and Sundays. Public holiday handling is not required in Phase 1.

**BR-8.5.5 — No Auto-Escalation:**
There is no automatic escalation mechanism in Phase 1. If a task is overdue (the due date has passed and the required action has not been completed), the system does not send reminder emails, reassign the task, or escalate to a manager. Overdue records can be identified through the audit table for manual management review. See Section 13.1 for this known limitation.

**BR-8.5.6 — Due Dates in Email Only:**
Task due dates are communicated exclusively through email notifications. There is no task calendar, task list interface, or due date indicator in the application UI in Phase 1.

---

### 8.6 Notification Rules

These rules define when email notifications are sent, to whom, and what they contain.

**BR-8.6.1 — Notification on Submission for Approval:**
When the CC Owner submits for approval (Initiated → Pending Implementation Approval), the system shall send an email to the assigned Approver. The email shall include the CC-ID, the Change Title, and the task due date (Submission Date + 5 business days).

**BR-8.6.2 — Notification on Implementation Approval:**
When the Approver approves at the Implementation gate (Pending Implementation Approval → In Implementation), the system shall send an email to the CC Owner. The email shall include the CC-ID, confirmation of approval, and the task due date for implementation (Target Closure Date − 3 business days).

**BR-8.6.3 — Notification on Implementation Rejection:**
When the Approver rejects at the Implementation gate (Pending Implementation Approval → Initiated), the system shall send an email to the CC Owner. The email shall include the CC-ID, notification of rejection, and instruction to revise and resubmit.

**BR-8.6.4 — Notification on Submission for Final Approval:**
When the CC Owner submits for final approval (In Implementation → Pending Final Approval), the system shall send an email to the assigned Approver. The email shall include the CC-ID, the Change Title, and the task due date (Target Closure Date).

**BR-8.6.5 — Notification on Final Approval (Closure):**
When the Approver approves at the Final gate (Pending Final Approval → Closed), the system shall send an email to the CC Owner. The email shall include the CC-ID and confirmation that the Change Control has been closed successfully.

**BR-8.6.6 — Notification on Final Rejection:**
When the Approver rejects at the Final gate (Pending Final Approval → In Implementation), the system shall send an email to the CC Owner. The email shall include the CC-ID, notification of rejection, and instruction to improve implementation documentation and resubmit.

**BR-8.6.7 — Notification on Cancellation:**
When the CC Owner cancels a CC (Initiated → Cancelled), the system shall send an email to the assigned Approver, if one was previously assigned. The email shall include the CC-ID and notification that the Change Control has been cancelled.

**BR-8.6.8 — No Direct Links in Emails:**
Email notifications shall include the CC-ID and a summary of the action or required task. They shall not contain direct clickable links to the CC record within the application. Users must navigate to the application and locate the record using the CC-ID.

**BR-8.6.9 — Email Templates:**
Each notification type shall use a dedicated email template with consistent branding and formatting. The specific template designs are an implementation detail and are not defined in this BRD.

**BR-8.6.10 — No Notifications to Viewers or Admins:**
Viewers and Admins do not receive workflow-related email notifications. Notifications are sent only to active workflow participants (CC Owner and assigned Approver) for the specific record.

---

### 8.7 Audit & Data Retention

These rules define how audit data is captured, stored, and retained.

**BR-8.7.1 — Automatic State Transition Logging:**
All state transitions shall be logged automatically in the audit table when a state change occurs. Each entry captures the from-state, to-state, performing user, and timestamp.

**BR-8.7.2 — Critical Field Change Logging:**
The following fields shall be logged in the audit table when their values change: Decision, Decision Comments, Risk Level, Final Decision, Final Comments, Cancellation Reason, Target Closure Date, Proposed Implementation Date, and Assign Approver. Each field change is logged as a separate audit entry with the old value and new value.

**BR-8.7.3 — Non-Critical Fields Not Logged:**
Edits to non-critical fields (e.g., Change Description, Business Impact, Risk Rationale, Implementation Summary, and other free-text content fields) are not individually tracked in the audit table. This keeps the audit log focused on significant, compliance-relevant changes.

**BR-8.7.4 — User Management Action Logging:**
Admin actions related to user management (user added, role changed, user deactivated) shall be logged in the audit table with the Admin's identity, the affected user, the action performed, and the timestamp.

**BR-8.7.5 — Granular Logging:**
Each critical field change shall be logged as a separate audit entry, even when multiple fields are updated in a single save or submission operation. Multiple entries from the same action shall share the same timestamp. For example, when an Approver submits a decision with values for Decision, Risk Level, and Decision Comments, the system creates 3 separate field-change audit entries plus 1 state transition entry, all with the same timestamp.

**BR-8.7.6 — Old Value Preservation on Overwrite:**
When a critical field value is overwritten (e.g., Decision changed from "Reject" to "Approve" during a re-review cycle), the audit entry shall capture both the old value and the new value. This ensures complete history is maintained even though the CC record itself only shows the latest values.

**BR-8.7.7 — Audit Records Immutable:**
Audit records shall never be deleted, modified, or overwritten. The audit log is an append-only, permanent record.

**BR-8.7.8 — CC Record Retention:**
Change Control records shall be retained in the system indefinitely. There is no automatic deletion, archival, or purging of CC records, including cancelled records.

**BR-8.7.9 — User Record Retention:**
User records shall be retained in the system even after deactivation. Deactivated users remain in the database so that audit trail entries and CC records that reference them continue to display the correct user names.

**BR-8.7.10 — Audit vs Application Logging:**
The audit table captures business-significant actions (what users did in the application). Technical API request/response logging for debugging and system monitoring is a separate concern and is independent of the business audit trail defined in this BRD.