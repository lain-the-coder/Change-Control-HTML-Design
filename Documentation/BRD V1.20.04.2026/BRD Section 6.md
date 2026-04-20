# Section 6 - Reviewed

## 6. FUNCTIONAL REQUIREMENTS

This section defines the functional requirements of the Change Control module, describing what the system must do across each major capability area. Requirements are identified with a prefix (FR-) for traceability. Each requirement is written from a business perspective without reference to specific technology stack or implementation approach.

---

### 6.1 Change Control Creation

### 6.1.1 Record Initiation

**FR-6.1.1:** The system shall allow users with the CC Owner role to create new Change Control records by clicking the "+ Create Change Control" button.

**FR-6.1.2:** The "+ Create Change Control" button shall be visible only to users with the CC Owner role. Users with Approver, Viewer, or Admin roles shall not see this button.

**FR-6.1.3:** On creation of a new Change Control, the system shall automatically generate and assign a unique CC-ID to the record. The CC-ID format shall follow the pattern CC-XXX (e.g., CC-001, CC-002, CC-003), with a sequential numeric portion. The CC-ID is permanent and cannot be changed.

**FR-6.1.4:** On creation, the system shall automatically populate the following fields:

- **CC-ID:** System-generated unique identifier
- **Current State:** Set to "Initiated"
- **Change Owner:** Set to the full name of the user who created the record
- **Last Updated By:** Set to the full name of the creating user
- **Created On:** Set to the current date and time
- **Last Updated On:** Set to the current date and time
- **Implementation Approval Status:** Set to "Not Submitted"
- **Final Approval Status:** Set to "Not Submitted"

**FR-6.1.5:** All system-generated fields populated at creation shall be read-only and cannot be modified by any user.

### 6.1.2 Form Structure at Creation

**FR-6.1.6:** The new CC form shall display all 50 fields organised into the following sections: Change Details (Identification, Change Definition, Planning), Impact & Risk Assessment, Implementation Plan & Validation, Implementation Details, Approvals (Initiation, Implementation Approval, Final Approval, Status), and Additional Information.

**FR-6.1.7:** In the Initiated state, the CC Owner shall have edit access to 25 fields as defined in the Security Matrix. All other fields shall be displayed as system-managed (read-only), not applicable (with contextual placeholder messages), or not yet populated (displayed as "—").

**FR-6.1.8:** Implementation Details fields (Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, Implementation Evidence) shall display as "Not applicable — Available after approval" in the Initiated state.

**FR-6.1.9:** Implementation Approval fields (Decision, Risk Level, Decision Comments) shall display as "Not applicable — Pending submission" or "Not applicable — Will be set by approver during review" in the Initiated state.

**FR-6.1.10:** Final Approval fields (Final Decision, Final Comments) shall display as "Not applicable — Pending implementation" in the Initiated state.

**FR-6.1.11:** The Cancellation Reason field shall be hidden in the Initiated state (and all states except Cancelled). It shall not appear on the form.

### 6.1.3 Save Draft

**FR-6.1.12:** The system shall provide a "Save Draft" function that saves all currently entered field values without triggering mandatory field validation.

**FR-6.1.13:** Saving a draft shall not change the record's state. The record shall remain in the Initiated state with all 25 fields still editable.

**FR-6.1.14:** On save, the system shall update the Last Updated By and Last Updated On fields to reflect the save action.

### 6.1.4 Submission for Approval

**FR-6.1.15:** The system shall provide a "Submit for Approval" button visible to the CC Owner in the Initiated state.

**FR-6.1.16:** On clicking "Submit for Approval," the system shall validate all mandatory fields. If any mandatory field is empty or contains only whitespace, the submission shall be blocked and the system shall display a validation error identifying each field that requires attention.

**FR-6.1.17:** On clicking "Submit for Approval," the system shall validate date fields as follows:

- The Proposed Implementation Date must be in the future (greater than the current date at the time of submission).
- The Proposed Implementation Date must be ≥ 2 business days from the current date.
- The Target Closure Date must be in the future (greater than the current date at the time of submission).
- The Target Closure Date must be ≥ 10 business days from the current date.
- If either date fails validation, the submission shall be blocked and the system shall display a specific validation error (e.g., "Proposed Implementation Date cannot be in the past. Please update.").

**FR-6.1.18:** If the user updates the dates after a validation failure, the system shall re-validate the dates against the new current date at the time of the subsequent submission attempt.

**FR-6.1.19:** On successful validation, the system shall transition the record from Initiated to Pending Implementation Approval. See Section 6.2 for the approval workflow requirements.

---

### 6.2 Approval Workflow

### 6.2.1 Approver Assignment

**FR-6.2.1:** The CC Owner shall assign an Approver during the Initiated state using the "Assign Approver" dropdown field.

**FR-6.2.2:** The "Assign Approver" dropdown shall only display users who hold the Approver role in the system. Users with CC Owner, Viewer, or Admin roles shall not appear in this dropdown.

**FR-6.2.3:** The system shall enforce segregation of duties. Since each user can hold only one role at a time, a CC Owner will never appear in the Approver dropdown (which only displays users with the Approver role). To prevent edge cases where a role change could create a conflict, the system shall block Admin from changing a user's role if that user has any active CC records (records in any state other than Closed or Cancelled) where they are either the CC Owner or the assigned Approver. See Section 8.4 (BR-8.4.11) for the detailed business rule.

**FR-6.2.4:** Once assigned, the same Approver shall be responsible for both approval gates (Implementation Approval and Final Approval) on that record.

### 6.2.2 Implementation Approval Gate (First Approval)

**FR-6.2.5:** When a record enters the Pending Implementation Approval state, the assigned Approver shall have edit access to exactly 3 fields: Decision (dropdown: Approve / Reject), Risk Level (dropdown: Low / Medium / High), and Decision Comments (text area).

**FR-6.2.5a:** When submitting a decision at the Implementation Approval gate, the Approver must populate all three fields: Decision (mandatory), Risk Level (mandatory), and Decision Comments (mandatory). The submission shall be blocked if any of these three fields is empty.

**FR-6.2.6:** All other fields shall be displayed as read-only to the Approver, showing the values submitted by the CC Owner.

**FR-6.2.7:** The system shall provide a single "Submit Decision" button to the assigned Approver. There shall be no separate "Approve" and "Reject" buttons.

**FR-6.2.8:** On clicking "Submit Decision," the system shall read the value of the Decision field to determine the state transition:

- If Decision = "Approve": the state transitions to In Implementation.
- If Decision = "Reject": the state transitions to Initiated (loop back).

**FR-6.2.9:** On approval (Decision = "Approve"), the system shall:

- Transition the state to In Implementation.
- Populate Implementation Approval By with the Approver's name (system-generated, read-only).
- Populate Implementation Approval On with the current date and time (system-generated, read-only).
- Update Implementation Approval Status to "Approved."
- Save the Decision, Risk Level, and Decision Comments values to the record.
- Send an email notification to the CC Owner with a task due date of Target Closure Date − 3 business days.
- Log the approval in the audit trail.

**FR-6.2.10:** On rejection (Decision = "Reject"), the system shall:

- Transition the state back to Initiated.
- Save the Decision ("Reject"), Risk Level (if set), and Decision Comments values to the record.
- Capture the Decision, Risk Level, and Decision Comments in the audit log before they can be overwritten during re-review.
- Send an email notification to the CC Owner informing them of the rejection and the need to revise and resubmit.
- Reset Implementation Approval Status to "Not Submitted."
- Log the rejection in the audit trail.

**FR-6.2.11:** The CC Owner's view of a record in Pending Implementation Approval shall show all fields as read-only. The "Submit Decision" button shall not be displayed to the CC Owner. The CC Owner waits for the Approver's decision.

### 6.2.3 Rejection and Resubmission at Implementation Approval

**FR-6.2.12:** When a record is rejected at the Implementation Approval gate and returns to the Initiated state, all 25 CC Owner-editable fields shall become editable again, including Target Closure Date.

**FR-6.2.13:** Previously entered field values shall be preserved when the record returns to Initiated. The CC Owner revises existing values rather than re-entering them from scratch.

**FR-6.2.14:** Date validations shall be re-applied at the time of resubmission. Both dates must be in the future and meet the minimum lead-time requirements based on the current date at the time of the new submission attempt.

**FR-6.2.15:** On resubmission, the standard "Submit for Approval" validation and transition process applies (see FR-6.1.15 through FR-6.1.19).

### 6.2.4 Final Approval Gate (Second Approval)

**FR-6.2.16:** When a record enters the Pending Final Approval state, the assigned Approver shall have edit access to exactly 2 fields: Final Decision (dropdown: Approve / Reject) and Final Comments (textarea).

**FR-6.2.16a:** When submitting a decision at the Final Approval gate, the Approver must populate both fields: Final Decision (mandatory) and Final Comments (mandatory). The submission shall be blocked if either field is empty.

**FR-6.2.17:** All other fields, including the 6 Implementation Details fields and the 3 Implementation Approval fields, shall be displayed as read-only to the Approver.

**FR-6.2.18:** The system shall provide a single "Submit Decision" button to the assigned Approver, identical in behaviour to the Implementation Approval gate.

**FR-6.2.19:** On clicking "Submit Decision," the system shall read the value of the Final Decision field to determine the state transition:

- If Final Decision = "Approve": the state transitions to Closed.
- If Final Decision = "Reject": the state transitions to In Implementation (loop back).

**FR-6.2.20:** On final approval (Final Decision = "Approve"), the system shall:

- Transition the state to Closed.
- Populate Final Approval By with the Approver's name (system-generated, read-only).
- Populate Final Approval On with the current date and time (system-generated, read-only).
- Populate Actual Closure Date with the current date and time (system-generated, read-only).
- Update Final Approval Status to "Approved."
- Save the Final Decision and Final Comments values to the record.
- Send an email notification to the CC Owner confirming the Change Control has been closed successfully.
- Log the final approval in the audit trail.

**FR-6.2.21:** On final rejection (Final Decision = "Reject"), the system shall:

- Transition the state back to In Implementation.
- Save the Final Decision ("Reject") and Final Comments values to the record.
- Capture the Final Decision and Final Comments in the audit log before they can be overwritten during re-review.
- Send an email notification to the CC Owner informing them that the implementation documentation needs improvement.
- Reset Final Approval Status to "Not Submitted."
- Log the rejection in the audit trail.

### 6.2.5 Rejection and Resubmission at Final Approval

**FR-6.2.22:** When a record is rejected at the Final Approval gate and returns to the In Implementation state, the 6 Implementation Details fields shall become editable for the CC Owner again.

**FR-6.2.23:** Previously entered implementation detail values shall be preserved for revision.

**FR-6.2.24:** The Approver's rejection feedback (Final Comments) shall remain visible to the CC Owner as a read-only field so they understand what needs improvement.

**FR-6.2.25:** On resubmission via "Submit for Final Approval," the standard transition to Pending Final Approval applies, and a new notification is sent to the Approver.

### 6.2.6 Approval Comments Behaviour

**FR-6.2.26:** Decision Comments (at the Implementation Approval gate) shall be used for both Approve and Reject decisions. There is no separate "Rejection Comments" field. The same field captures the Approver's rationale regardless of the decision.

**FR-6.2.27:** Final Comments (at the Final Approval gate) shall follow the same behaviour: used for both Approve and Reject decisions.

**FR-6.2.28:** When an Approver re-reviews a record after a rejection cycle, the new Decision Comments or Final Comments overwrite the previous values in the record. The old values are preserved in the audit log before overwrite, maintaining a complete comment history across review cycles.

**FR-6.2.29: Role Change Restriction for Active Records**

The system shall prevent an Admin from changing a user's role if that user is associated with any active Change Control records. A record is considered "active" if it is in any state other than Closed or Cancelled.

The restriction applies when the user is:

- The CC Owner (creator) of an active record, OR
- The assigned Approver on an active record

When the Admin attempts to change a user's role and active records exist, the system shall:

1. Block the role change.
2. Display an error message identifying the active CC-IDs preventing the change (e.g., "Cannot change role: User is associated with active records CC-001, CC-003. These records must be Closed or Cancelled before the role change can proceed.").

This requirement prevents segregation of duties violations that could occur if a CC Owner's role were changed to Approver while they have active records.

---

### 6.3 Implementation Tracking

### 6.3.1 Implementation State

**FR-6.3.1:** When a record enters the In Implementation state (via approval at the Implementation Approval gate or via rejection at the Final Approval gate), the CC Owner shall have edit access to 6 fields: Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, and Implementation Evidence.

**FR-6.3.2:** All original change detail fields (Change Definition, Planning, Impact & Risk Assessment, Implementation Plan & Validation) shall be displayed as read-only in this state.

**FR-6.3.3:** The Implementation Approval fields (Decision, Risk Level, Decision Comments, Implementation Approval By, Implementation Approval On) shall be displayed as read-only showing the approved values.

**FR-6.3.4:** Implementation Approval Status shall display "Approved" and Final Approval Status shall display "Not Submitted."

**FR-6.3.5:** The system shall provide a "Submit for Final Approval" button to the CC Owner in the In Implementation state.

### 6.3.2 Implementation Evidence Upload

**FR-6.3.6:** The Implementation Evidence field shall support file upload functionality, allowing the CC Owner to attach evidence of the completed implementation.

**FR-6.3.6a:** The Implementation Evidence field is mandatory. The CC Owner must upload an evidence file before submitting for final approval.

**FR-6.3.7:** Supported file types for Implementation Evidence shall be the same as for Supporting Documents: PDF, DOCX, XLSX, and image files (PNG, JPG).

**FR-6.3.8:** Maximum file size shall be 10MB per file.

**FR-6.3.9:** Each file upload field supports a single file upload. Users should combine related documents into one file before uploading.

---

### 6.4 Notifications & Task Management

### 6.4.1 Notification Delivery

**FR-6.4.1:** The system shall send email notifications to relevant users at each state transition. Notifications shall be the primary mechanism for informing users of required actions.

**FR-6.4.2:** Each email notification shall include the CC-ID and a summary of the required action. Notifications shall not contain direct clickable links to the CC record within the application.

**FR-6.4.3:** Email notifications shall use templates appropriate to each notification type, providing clear and consistent communication.

### 6.4.2 Notification Triggers

The following table defines all notification triggers, recipients, and content:

| # | Trigger Event | Recipient | Notification Content | Task Due Date |
| --- | --- | --- | --- | --- |
| N1 | CC Owner clicks "Submit for Approval" (Initiated → Pending Implementation Approval) | Assigned Approver | New CC submitted for your review. Review and submit your decision. | Submission Date + 5 business days |
| N2 | Approver Decision = Approve (Pending Implementation Approval → In Implementation) | CC Owner | Your CC has been approved for implementation. Begin implementation and document details. | Target Closure Date − 3 business days |
| N3 | Approver Decision = Reject (Pending Implementation Approval → Initiated) | CC Owner | Your CC has been rejected. Review the Approver's comments, revise, and resubmit. | None |
| N4 | CC Owner clicks "Submit for Final Approval" (In Implementation → Pending Final Approval) | Assigned Approver | Implementation complete. Review implementation evidence and submit your final decision. | Target Closure Date |
| N5 | Approver Final Decision = Approve (Pending Final Approval → Closed) | CC Owner | Your CC has been approved and closed successfully. | None |
| N6 | Approver Final Decision = Reject (Pending Final Approval → In Implementation) | CC Owner | Final approval rejected. Improve implementation documentation and resubmit. | None |
| N7 | CC Owner cancels CC (Initiated → Cancelled) | Assigned Approver (if one was assigned) | CC has been cancelled by the CC Owner. | None |

### 6.4.3 Task Due Dates

**FR-6.4.4:** Task due dates are communicated via email notifications only. There is no task calendar, task list UI, or task management interface in Phase 1.

**FR-6.4.5:** Task due dates are calculated as follows:

**Task 1 — Implementation Approval Review:**

- Assignee: Approver
- Due Date: Submission Date + 5 business days
- Triggered when: Record transitions to Pending Implementation Approval (Notification N1)

**Task 2 — Implementation Completion:**

- Assignee: CC Owner
- Due Date: Target Closure Date − 3 business days
- Triggered when: Record transitions to In Implementation via approval (Notification N2)

**Task 3 — Final Approval Review:**

- Assignee: Approver
- Due Date: Target Closure Date
- Triggered when: Record transitions to Pending Final Approval (Notification N4)

**FR-6.4.6:** Business days calculations shall exclude weekends (Saturday and Sunday). Public holiday handling is not required in Phase 1.

**FR-6.4.7:** There is no auto-escalation, reminder email, or overdue notification mechanism in Phase 1. If a task is overdue, it is tracked through the audit table for manual management review (see Section 13.1 for known limitations).

---

### 6.5 Cancellation

### 6.5.1 Cancellation Eligibility

**FR-6.5.1:** Cancellation is available only when the record is in the Initiated state. Once a record has been submitted for approval or is in any subsequent state (Pending Implementation Approval, In Implementation, Pending Final Approval, Closed), it cannot be cancelled.

**FR-6.5.2:** Only the CC Owner of that specific record can cancel it. Other users with the CC Owner role cannot cancel a record they did not create. Users with Approver, Viewer, or Admin roles cannot cancel any record.

**FR-6.5.3:** The "Cancel CC" button shall be visible only when both conditions are met: the record is in the Initiated state, and the logged-in user is the CC Owner of that record.

### 6.5.2 Cancellation Workflow

**FR-6.5.4:** When the CC Owner clicks "Cancel CC," the system shall display a popup modal with the following elements:

- Title: "Cancel Change Control"
- Confirmation message: "Are you sure you want to cancel CC-[ID]? This action cannot be undone."
- Cancellation Reason field: Text area, mandatory, maximum 500 characters
- "Confirm Cancellation" button (red/danger styling)
- "Go Back" button (grey/secondary styling)

**FR-6.5.5:** The Cancellation Reason field in the popup modal is mandatory. The system shall not allow confirmation if the field is empty or contains only whitespace.

**FR-6.5.6:** Clicking "Go Back" shall close the popup modal and return the user to the CC form with no changes made.

**FR-6.5.7:** On clicking "Confirm Cancellation" with a valid Cancellation Reason:

1. The state transitions from Initiated to Cancelled.
2. The Cancellation Reason is saved to the CC record as field #50.
3. All fields become read-only for all roles.
4. The Cancellation Reason field becomes visible in the Additional Information section, below the Comments field, displayed as a read-only textarea.
5. Implementation Approval Status is set to "N/A."
6. Final Approval Status is set to "N/A."
7. If an Approver was previously assigned to the record, an email notification is sent to inform them of the cancellation.
8. The audit log captures the state transition (Initiated → Cancelled) and the Cancellation Reason value.

### 6.5.3 Cancellation Permanence

**FR-6.5.8:** Cancellation is a permanent, irreversible action. There is no "reopen," "reactivate," or "undo cancellation" function. Once cancelled, the record remains in the Cancelled state indefinitely.

**FR-6.5.9:** Cancelled records are retained in the system permanently. They are not deleted, archived, or hidden. They remain visible and accessible in the All Change Controls list and can be opened and viewed by any user.

**FR-6.5.10:** No workflow action buttons shall be displayed on a Cancelled record for any role.

---

### 6.6 Audit Trail & History

### 6.6.1 Overview

**FR-6.6.1:** The system shall maintain a comprehensive audit log to track all significant actions and changes within the Change Control module. The audit trail provides a permanent, tamper-proof record of who did what, when, and what changed.

**FR-6.6.2:** The audit trail shall be stored in a database table. There is no user interface for viewing the audit log in Phase 1. A future enhancement will add an audit history viewer within the CC form (see Section 13.2).

**FR-6.6.3:** Audit records shall never be deleted, modified, or overwritten. The audit log is an append-only record.

### 6.6.2 Auditable Events

The system shall log the following categories of events:

**Change Control Actions:**

- CC creation (new record created)
- All state transitions (who triggered the transition, when, from which state, to which state)
- Critical field updates only — the following fields are tracked individually:
    - Decision and Decision Comments
    - Risk Level
    - Final Decision and Final Comments
    - Cancellation Reason
    - Target Closure Date (initial value set and any subsequent changes)
    - Proposed Implementation Date (if changed)
    - Assign Approver (if changed)

**User Management Actions (Admin):**

- User added to the system (including assigned role)
- User role changed (old role and new role)
- User deactivated or deleted

**Non-critical field changes are not logged.** Edits to fields such as Change Description, Business Impact, Risk Rationale, and other free-text content fields are not individually tracked in the audit log. This keeps the audit table focused on significant, compliance-relevant changes.

### 6.6.3 Audit Log Structure

Each audit entry shall capture the following information:

| Field | Description |
| --- | --- |
| **Audit ID** | Unique identifier for the audit entry (auto-generated) |
| **Entity Type** | The type of entity being audited (e.g., "ChangeControl", "User", "SystemSetting") |
| **Entity ID** | The identifier of the specific entity (e.g., "CC-001", "User-123") |
| **Action Type** | The category of action performed (e.g., "Created", "StateChanged", "FieldUpdated", "UserAdded", "UserRoleChanged") |
| **Action Description** | A human-readable summary of the action, generated by the system at runtime (e.g., "State changed from Initiated to Pending Implementation Approval") |
| **Field Name** | The name of the field that was changed (null for non-field actions such as record creation) |
| **Old Value** | The previous value of the field before the change (null for creation events) |
| **New Value** | The new value of the field after the change |
| **Performed By** | The user ID of the person who performed the action |
| **Performed By Name** | The display name of the person who performed the action |
| **Timestamp** | The date and time when the action occurred |

### 6.6.4 Audit Log Examples

The following examples illustrate how audit entries are captured for different scenarios:

**Example 1 — CC Created:**
Entity Type: ChangeControl | Entity ID: CC-001 | Action Type: Created | Action Description: "Change Control CC-001 created" | Field Name: — | Old Value: — | New Value: CC-001 | Performed By Name: John Doe | Timestamp: 2026-04-14 10:00:00

**Example 2 — State Changed (Submission):**
Entity Type: ChangeControl | Entity ID: CC-001 | Action Type: StateChanged | Action Description: "State changed from Initiated to Pending Implementation Approval" | Field Name: Current State | Old Value: Initiated | New Value: Pending Implementation Approval | Performed By Name: John Doe (CC Owner) | Timestamp: 2026-04-14 11:00:00

**Example 3 — Field Updated (Decision Set):**
Entity Type: ChangeControl | Entity ID: CC-001 | Action Type: FieldUpdated | Action Description: "Decision set to Approve" | Field Name: Decision | Old Value: — | New Value: Approve | Performed By Name: Jane Smith (Approver) | Timestamp: 2026-04-15 14:00:00

**Example 4 — Field Updated (Decision Comments):**
Entity Type: ChangeControl | Entity ID: CC-001 | Action Type: FieldUpdated | Action Description: "Decision Comments added" | Field Name: Decision Comments | Old Value: — | New Value: "All requirements met. Approved for implementation." | Performed By Name: Jane Smith (Approver) | Timestamp: 2026-04-15 14:00:00

**Example 5 — Cancellation Reason Captured:**
Entity Type: ChangeControl | Entity ID: CC-002 | Action Type: FieldUpdated | Action Description: "Cancellation Reason provided" | Field Name: Cancellation Reason | Old Value: — | New Value: "Business requirements changed, no longer needed" | Performed By Name: John Doe (CC Owner) | Timestamp: 2026-04-16 10:00:00

**Example 6 — User Added (Admin Action):**
Entity Type: User | Entity ID: User-789 | Action Type: UserAdded | Action Description: "User Bob Johnson added with role CC Owner" | Field Name: — | Old Value: — | New Value: "Bob Johnson - CC Owner" | Performed By Name: Admin User | Timestamp: 2026-04-14 08:00:00

**Example 7 — User Role Changed:**
Entity Type: User | Entity ID: User-789 | Action Type: UserRoleChanged | Action Description: "User Bob Johnson role changed" | Field Name: Role | Old Value: CC Owner | New Value: Approver | Performed By Name: Admin User | Timestamp: 2026-04-20 09:00:00

### 6.6.5 Rejection History Preservation

**FR-6.6.4:** When approval or rejection fields are overwritten during a re-review cycle (after a previous rejection), the system shall capture the old values in the audit log before the overwrite occurs. This ensures a complete rejection history is maintained even though the CC record itself only displays the latest values.

**Rejection and Re-Approval Scenario:**

*First Review — Rejection:*

- Audit Entry: Decision changed from [empty] to "Reject"
- Audit Entry: Risk Level changed from [empty] to "Medium"
- Audit Entry: Decision Comments changed from [empty] to "Need more details on risk mitigation"
- Audit Entry: State changed from "Pending Implementation Approval" to "Initiated"

*After CC Owner revises and resubmits — Second Review — Approval:*

- Audit Entry: Decision changed from "Reject" to "Approve" — **old value "Reject" preserved**
- Audit Entry: Risk Level changed from "Medium" to "Low" — **old value "Medium" preserved**
- Audit Entry: Decision Comments changed from "Need more details on risk mitigation" to "Risk mitigation plan now adequate. Approved." — **old rejection comment preserved**
- Audit Entry: State changed from "Pending Implementation Approval" to "In Implementation"

The CC record now shows Decision = "Approve", Risk Level = "Low", and the latest Decision Comments. But the audit log retains the complete history showing the initial rejection, its rationale, and the subsequent approval.

### 6.6.6 Logging Granularity

**FR-6.6.5:** Each critical field change shall be logged as a separate audit entry, even when multiple fields are updated in a single save or submission operation. Multiple entries from the same action shall share the same timestamp.

**Example:** When an Approver submits a decision with Decision = "Approve", Risk Level = "Low", and Decision Comments = "Looks good", the system creates 3 separate audit entries (one for each field) plus 1 state transition entry, all with the same timestamp.

**FR-6.6.6:** Non-critical field changes (e.g., editing Change Description text, updating Business Impact narrative) shall not generate audit entries. The audit table captures only significant, compliance-relevant changes as defined in Section 6.6.2.

### 6.6.7 Audit Data Retention

**FR-6.6.7:** All audit records shall be retained indefinitely. There is no expiry, archival, or automatic purging of audit data.

**FR-6.6.8:** Change Control records shall be retained indefinitely. There is no automatic deletion of CC records, including cancelled records.

**FR-6.6.9:** User records shall be retained even after deactivation. Deactivated users remain in the database for audit trail referencing.

**FR-6.6.10:** The audit table captures business-significant actions (what users did in the application). Technical API request/response logging for debugging and system monitoring is a separate concern and is not covered by this audit trail requirement.

---