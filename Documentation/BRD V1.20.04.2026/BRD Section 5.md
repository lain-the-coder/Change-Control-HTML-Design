# Section 5 - Reviewed

## 5. USER STORIES & USE CASES

This section defines the user stories for each role, describing what each user needs to accomplish within the Change Control module. Each story follows the format: *"As a [role], I want to [action], so that [business value]."* Acceptance criteria are provided for each story to define the conditions that must be met for the story to be considered complete.

---

### 5.1 CC Owner Stories

### US-CC-01: Create a New Change Control

**As a** CC Owner, **I want to** create a new Change Control record, **so that** I can formally document a proposed change and initiate the approval process.

**Acceptance Criteria:**

1. The "+ Create Change Control" button is visible on the Dashboard and accessible from the navigation only to users with the CC Owner role.
2. Clicking the button opens a new CC form in the Initiated state with a system-generated CC-ID.
3. The Change Owner field is automatically populated with my name and is read-only.
4. Created On and Last Updated On are automatically populated with the current date and time.
5. Last Updated By is automatically populated with who modified the record last.
6. All 25 user-editable fields are displayed as active input controls.
7. Implementation Details fields display as "Not applicable — Available after approval."
8. Approval fields display as "Not applicable — Pending submission" or similar contextual placeholders.
9. Implementation Approval Status displays "Not Submitted" and Final Approval Status displays "Not Submitted."
10. The "Save Draft," "Submit for Approval," and "Cancel CC" buttons are available.

---

### US-CC-02: Save a Draft Change Control

**As a** CC Owner, **I want to** save my Change Control as a draft without submitting it, **so that** I can return to it later and continue filling in the details.

**Acceptance Criteria:**

1. Clicking "Save Draft" saves all currently entered field values without triggering mandatory field validation.
2. The record remains in the Initiated state with all 25 fields still editable.
3. The Last Updated On and Last Updated By fields are updated to reflect the save action.
4. The saved record appears in the My Change Controls list and the All Change Controls list with the status "Initiated."
5. I can reopen the saved draft and continue editing at any time.

---

### US-CC-03: Submit a Change Control for Approval

**As a** CC Owner, **I want to** submit my completed Change Control for implementation approval, **so that** the assigned Approver can review and evaluate the proposed change.

**Acceptance Criteria:**

1. Clicking "Submit for Approval" triggers validation of all mandatory fields. If any mandatory field is empty, the submission is blocked and a validation error is displayed identifying the missing fields.
2. The Proposed Implementation Date is validated to be ≥ 2 business days from the current date at the time of submission. If it is in the past or less than 2 business days away, the submission is blocked with a validation error.
3. The Target Closure Date is validated to be ≥ 10 business days from the current date at the time of submission. If it is in the past or less than 10 business days away, the submission is blocked with a validation error.
4. On successful validation, the state transitions from Initiated to Pending Implementation Approval.
5. An email notification is sent to the assigned Approver with a task due date of Submission Date + 5 business days.
6. All 25 previously editable fields become read-only for me.
7. Implementation Approval Status changes from "Not Submitted" to "Pending."
8. I can still view the record but cannot edit any fields.

---

### US-CC-04: Cancel a Change Control

**As a** CC Owner, **I want to** cancel a Change Control that I created while it is still in the Initiated state, **so that** I can formally terminate a change request that is no longer needed.

**Acceptance Criteria:**

1. The "Cancel CC" button is visible only when the record is in the Initiated state and only to me as the owner of that specific record.
2. Clicking "Cancel CC" displays a popup modal with the title "Cancel Change Control," a confirmation message including the CC-ID, a mandatory Cancellation Reason text area (max 500 characters), a "Confirm Cancellation" button (red), and a "Go Back" button (grey).
3. I cannot confirm the cancellation without entering a Cancellation Reason. Empty or whitespace-only values are rejected.
4. On confirmation, the state transitions from Initiated to Cancelled permanently.
5. The Cancellation Reason is saved as field #50 on the record.
6. All fields become read-only.
7. The Cancellation Reason is displayed in the Additional Information section below the Comments field, visible as read-only.
8. Implementation Approval Status and Final Approval Status both display "N/A."
9. An email notification is sent to the assigned Approver (if one was selected).
10. The audit log captures the state transition and the Cancellation Reason.
11. The cancellation cannot be undone. There is no "reopen" or "reactivate" action.

---

### US-CC-05: Revise and Resubmit After Rejection at Implementation Approval

**As a** CC Owner, **I want to** revise my Change Control after it has been rejected at the Implementation Approval gate and resubmit it, **so that** I can address the Approver's feedback and continue the approval process.

**Acceptance Criteria:**

1. When the record is rejected, I receive an email notification informing me that the record needs revision.
2. The record returns to the Initiated state and all 25 fields become editable again.
3. Previously entered values are preserved — I revise what needs to change, not re-create from scratch.
4. The Approver's rejection comments (Decision Comments) are visible to me so I can understand what needs to be addressed.
5. I can update any of the 25 editable fields as needed.
6. Date validations are re-applied at the time of resubmission (both dates must still be in the future and meet the minimum lead-time requirements based on the new current date).
7. On resubmission, the state transitions back to Pending Implementation Approval and a new notification is sent to the Approver.

---

### US-CC-06: Complete Implementation Details

**As a** CC Owner, **I want to** document the implementation details after my change has been approved, **so that** I can provide evidence that the change was executed as planned and is ready for final review.

**Acceptance Criteria:**

1. When the record is in the In Implementation state, I have edit access to 6 fields: Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, and Implementation Evidence (file upload).
2. All original change detail fields are displayed as read-only showing the values I submitted.
3. The Approver's decision fields (Decision, Risk Level, Decision Comments) are displayed as read-only showing the approved values.
4. Implementation Approval By and Implementation Approval On display the Approver's name and the timestamp of approval.
5. The "Submit for Final Approval" button is available.

---

### US-CC-07: Submit for Final Approval

**As a** CC Owner, **I want to** submit my completed implementation for final approval, **so that** the Approver can verify the implementation was successful and close the Change Control.

**Acceptance Criteria:**

1. Clicking "Submit for Final Approval" transitions the state from In Implementation to Pending Final Approval.
2. An email notification is sent to the assigned Approver with a task due date of Target Closure Date.
3. All fields, including the 6 implementation detail fields, become read-only for me.
4. Final Approval Status changes from "Not Submitted" to "Pending."

---

### US-CC-08: Revise and Resubmit After Rejection at Final Approval

**As a** CC Owner, **I want to** revise my implementation details after the final approval has been rejected, **so that** I can improve the implementation documentation and resubmit for final review.

**Acceptance Criteria:**

1. When the final approval is rejected, I receive an email notification informing me that the implementation documentation needs improvement.
2. The record returns to the In Implementation state and the 6 implementation detail fields become editable again.
3. Previously entered implementation values are preserved for revision.
4. The Approver's rejection feedback (Final Comments) is visible to me.
5. On resubmission, the state transitions back to Pending Final Approval and a new notification is sent to the Approver.

---

### US-CC-09: View All Change Controls

**As a** CC Owner, **I want to** view all Change Controls in the system (not just my own), **so that** I can see the overall change activity and find records relevant to my work.

**Acceptance Criteria:**

1. The All Change Controls list view is accessible from the navigation.
2. The list displays all CC records in the system regardless of ownership.
3. I can view any record by clicking on it, though my edit access depends on whether I am the owner and what state the record is in.

---

### US-CC-10: View My Change Controls

**As a** CC Owner, **I want to** see a filtered list of only the Change Controls I own, **so that** I can quickly find and manage my own records.

**Acceptance Criteria:**

1. The My Change Controls list view is accessible from the navigation.
2. The list displays only CC records where I am the Change Owner.
3. Each record shows the CC-ID, Change Title, Current State, and last updated date.

---

### 5.2 Approver Stories

### US-AP-01: View Pending Approvals Queue

**As an** Approver, **I want to** see a queue of Change Controls that are pending my review, **so that** I can prioritise and manage my approval workload.

**Acceptance Criteria:**

1. The Approvals view is accessible from the navigation.
2. The list displays only CC records where I am the assigned Approver and the record is in a state requiring my action (Pending Implementation Approval or Pending Final Approval).
3. Each record shows the CC-ID, Change Title, CC Owner, Current State, and date submitted.
4. The Dashboard also displays a "Pending Approvals" card showing the count and list of items awaiting my decision.

---

### US-AP-02: Review and Approve at Implementation Approval Gate

**As an** Approver, **I want to** review a submitted Change Control, assess the risk, and approve it for implementation, **so that** the CC Owner can proceed with implementing the change.

**Acceptance Criteria:**

1. When I open a record in the Pending Implementation Approval state, all CC Owner-submitted fields are displayed as read-only so I can review the full change request.
2. I have edit access to exactly 3 fields: Decision (dropdown: Approve/Reject), Risk Level (dropdown: Low/Medium/High), and Decision Comments (text area).
3. I set the Decision field to "Approve," set the Risk Level, and enter Decision Comments.
4. Clicking "Submit Decision" transitions the state to In Implementation.
5. Implementation Approval By is populated with my name and Implementation Approval On is populated with the current timestamp.
6. An email notification is sent to the CC Owner with a task due date of Target Closure Date − 3 business days.
7. Implementation Approval Status changes from "Pending" to "Approved."

---

### US-AP-03: Review and Reject at Implementation Approval Gate

**As an** Approver, **I want to** reject a submitted Change Control that does not meet the required standard, **so that** the CC Owner can revise and improve it before implementation begins.

**Acceptance Criteria:**

1. I set the Decision field to "Reject" and enter Decision Comments explaining the reason for rejection.
2. Clicking "Submit Decision" transitions the state from Pending Implementation Approval back to Initiated.
3. The audit log captures my Decision ("Reject"), Risk Level (if set), and Decision Comments.
4. An email notification is sent to the CC Owner informing them of the rejection and the need to revise and resubmit.
5. Implementation Approval Status returns to "Not Submitted" (the record is back in Initiated).

---

### US-AP-04: Review and Approve at Final Approval Gate

**As an** Approver, **I want to** review the implementation evidence and approve the final closure of the Change Control, **so that** the change is formally completed and the record is closed.

**Acceptance Criteria:**

1. When I open a record in the Pending Final Approval state, all fields including the 6 implementation details are displayed as read-only for my review.
2. I have edit access to exactly 2 fields: Final Decision (dropdown: Approve/Reject) and Final Comments (text area).
3. I set the Final Decision to "Approve" and enter Final Comments.
4. Clicking "Submit Decision" transitions the state to Closed.
5. Final Approval By is populated with my name and Final Approval On is populated with the current timestamp.
6. Actual Closure Date is system-generated with the current date and time.
7. An email notification is sent to the CC Owner confirming the Change Control has been closed successfully.
8. Implementation Approval Status displays "Approved" and Final Approval Status displays "Approved."

---

### US-AP-05: Review and Reject at Final Approval Gate

**As an** Approver, **I want to** reject the implementation if the evidence or documentation is insufficient, **so that** the CC Owner can improve the implementation details before the record is closed.

**Acceptance Criteria:**

1. I set the Final Decision field to "Reject" and enter Final Comments explaining what needs improvement.
2. Clicking "Submit Decision" transitions the state from Pending Final Approval back to In Implementation.
3. The audit log captures my Final Decision ("Reject") and Final Comments.
4. An email notification is sent to the CC Owner informing them that the implementation documentation needs improvement.
5. Final Approval Status returns to "Not Submitted" (the record is back in In Implementation).

---

### US-AP-06: Re-Review After CC Owner Revision

**As an** Approver, **I want to** re-review a Change Control that I previously rejected after the CC Owner has revised it, **so that** I can evaluate whether the issues have been addressed.

**Acceptance Criteria:**

1. When a previously rejected record is resubmitted, I receive an email notification.
2. I can see the revised content in the record.
3. The same editable fields are available to me as during the original review (3 fields at Implementation Approval gate, 2 fields at Final Approval gate).
4. My new Decision, Risk Level, and Comments overwrite the previous rejection values on the record.
5. The audit log preserves the old rejection values before they are overwritten, maintaining a complete review history.

---

### 5.3 Viewer Stories

### US-VI-01: View All Change Controls

**As a** Viewer, **I want to** view all Change Controls in the system, **so that** I can monitor change activity and stay informed about ongoing and completed changes.

**Acceptance Criteria:**

1. I can access the All Change Controls list view from the navigation.
2. The list displays all CC records in the system across all states.
3. I can open any record and see all fields and sections in read-only mode.
4. No edit controls, workflow action buttons, or submission buttons are displayed to me.
5. Zero fields are editable for me in any state.

---

### US-VI-02: View Dashboard Overview

**As a** Viewer, **I want to** see the Dashboard with system-wide statistics, **so that** I can get a quick overview of Change Control activity across the organisation.

**Acceptance Criteria:**

1. The Dashboard displays Overview statistics showing the count of records in each state (Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed).
2. The "Pending Approvals" and "My Drafts" cards show empty states (since I do not participate in the workflow).
3. The "+ Create Change Control" button is not visible to me.

---

### US-VI-03: View Individual Change Control Detail

**As a** Viewer, **I want to** open and read a specific Change Control record in full detail, **so that** I can review the change documentation, decisions, and implementation evidence for audit or stakeholder purposes.

**Acceptance Criteria:**

1. I can open any CC record from the All Change Controls list.
2. All form sections and fields are displayed with their current values.
3. Fields that are "Not applicable" at the record's current state display the same placeholder messages as other roles see.
4. All fields are read-only. No input controls are active.
5. No action buttons (Submit, Cancel, Submit Decision) are displayed.

---

### 5.4 Admin Stories

### US-AD-01: Create a New User

**As an** Admin, **I want to** create new user accounts in the system, **so that** team members can access the Change Control module with the appropriate role.

**Acceptance Criteria:**

1. The Settings page includes a "Create New User" section visible only to Admins.
2. I can enter the user's full name, email address, password, and select a role from the four available roles (CC Owner, Approver, Viewer, Admin).
3. On creation, the new user can immediately sign in with the credentials I set.
4. The audit log captures the user creation event including the assigned role.
5. The new user appears in the All Users table.

---

### US-AD-02: Manage User Roles

**As an** Admin, **I want to** change a user's role, **so that** I can adjust permissions when team members' responsibilities change.

**Acceptance Criteria:**

1. The All Users table in Settings displays each user's name, email, current role, and action buttons.
2. I can edit a user's Full Name.
3. I can change a user's role by selecting a new role from the available options.
4. The system shall block the role change if the user has any active CC records (records in any state other than Closed or Cancelled) where they are either the CC Owner or the assigned Approver. The system shall display an error message identifying the active CC-IDs that must be resolved before the role change can proceed.
5. If no active records exist, the role change takes effect immediately for the user's future actions.
6. The audit log captures the role change event including the old role and new role.
7. Existing Closed or Cancelled CC records owned by or assigned to the user continue to display their name as the Change Owner or Approver — these historical references are not affected by the role change.
8. I cannot edit a user's email address (set at creation only).
9. I cannot reset a user's password through the application (password resets are handled through the Forgot Password flow or at the database level).

---

### US-AD-03: Deactivate a User

**As an** Admin, **I want to** deactivate a user account, **so that** I can revoke access for users who no longer need it while preserving their historical data.

**Acceptance Criteria:**

1. I can deactivate a user from the All Users table.
2. A deactivated user can no longer sign in to the system.
3. The user's historical data is preserved — CC records they created or approved still reference their name.
4. The user record is retained in the system (not deleted) for audit purposes.
5. The audit log captures the deactivation event.

---

### US-AD-04: View All Change Controls (Read-Only)

**As an** Admin, **I want to** view all Change Controls in the system, **so that** I can monitor the overall change management activity.

**Acceptance Criteria:**

1. I can access the All Change Controls list view and open any record.
2. All fields are displayed as read-only, identical to the Viewer experience.
3. No workflow action buttons are displayed to me on CC records.
4. Zero fields are editable for me on CC records in any state.
5. The "+ Create Change Control" button is not visible to me.

---

### US-AD-05: Edit User Profile

**As an** Admin, **I want to** edit a user's full name, **so that** I can keep user records accurate when names change (e.g., legal name change, name correction).

**Acceptance Criteria:**

1. I can edit the Full Name field for any user in the All Users table.
2. The updated name is reflected across the system wherever the user's name is displayed (e.g., Change Owner field on new records, Approver dropdown, user lists).
3. Historical references in existing CC records (Change Owner, Approver names in closed/cancelled records) may or may not be updated — this is an implementation decision. The BRD does not prescribe retroactive name updates on historical records.
4. The audit log captures the name change with old and new values.