# Section 3 - Reviewed

---

## 3. WORKFLOW & STATES

### 3.1 State Machine Diagram

The following diagram illustrates the complete Change Control lifecycle, showing all six states, transition paths, rejection loops, and the actions that trigger each transition:

![Flow Chart.png](Section%203%20-%20Reviewed/Flow_Chart.png)

---

### 3.2 Sequence Diagram

The following diagram illustrates the interaction between the CC Owner, the System, and the Approver across the full Change Control lifecycle, including both rejection scenarios:

![Sequence Diagram.png](Section%203%20-%20Reviewed/Sequence_Diagram.png)

---

### 3.3 State Descriptions

The Change Control module operates on a six-state lifecycle. Each state represents a distinct phase in the change management process with defined responsibilities, editable fields, and available actions.

### 3.3.1 Initiated

**Purpose:** The Initiated state is the creation and preparation phase. The CC Owner documents the change request by filling out all required fields before submitting for approval.

**Who Has Edit Access:** CC Owner (25 editable fields)

**Entry Conditions:**

- A user with the CC Owner role clicks the "+ Create Change Control" button, OR
- A previously submitted record is rejected at the Implementation Approval gate and returned to this state

**Editable Fields (25):**
Change Title, Change Description, Change Type, Change Category, Department/Function, Affected Systems/Modules, Proposed Implementation Date, Target Closure Date, Implementation Window Start, Implementation Window End, Reason for Change, Business Impact, Expected Downtime, Requires Testing, Requires Training, Risk Rationale, Key Risks & Mitigations, Supporting Documents, High-Level Implementation Plan, Validation Approach, Success Criteria, Rollback/Backout Plan, Assign Approver, Comments for Approver, Comments.

**Available Actions:**

- **Save Draft:** CC Owner can save the record without submitting. The record remains in Initiated state with all fields still editable.
- **Submit for Approval:** CC Owner submits the record for implementation approval. All mandatory fields must pass validation. Both date fields must be in the future at the time of submission. On success, the state transitions to Pending Implementation Approval.
- **Cancel CC:** CC Owner can cancel the record. A popup modal requires a mandatory Cancellation Reason (max 500 characters). On confirmation, the state transitions to Cancelled. This action is permanent and cannot be undone.

**Field Display Behaviour:**

- The 25 editable fields are shown as active input controls (text inputs, dropdowns, textareas, date pickers, file upload).
- The 6 Identification fields (CC-ID, Current State, Change Owner, Last Updated By, Created On, Last Updated On) are displayed as system-managed read-only values.
- Implementation Details fields (6 fields) are displayed as "Not applicable — Available after approval."
- Implementation Approval fields (Decision, Risk Level, Decision Comments) are displayed as "Not applicable — Pending submission" or "Not applicable — Will be set by approver during review."
- Final Approval fields are displayed as "Not applicable — Pending implementation."
- System timestamp and approval-tracking fields show "—" (dash) indicating no value yet.
- Implementation Approval Status: "Not Submitted"
- Final Approval Status: "Not Submitted"

**Rejection Re-Entry Behaviour:**
When a record returns to Initiated after rejection at the Implementation Approval gate, the following applies:

- All 25 fields become editable again for the CC Owner to revise.
- Previously entered values are preserved (the CC Owner revises, not re-creates).
- *The Target Closure Date is editable because the record is in the Initiated state, and the Security Matrix grants the CC Owner edit access to all 25 fields in this state.*
- The previous Decision, Risk Level, and Decision Comments values from the rejected review are overwritten when the Approver re-reviews. The old values are preserved in the audit log.

---

### 3.3.2 Pending Implementation Approval

**Purpose:** The record is awaiting the assigned Approver's review. The Approver evaluates the change request for completeness, risk, and readiness, then makes an Approve or Reject decision.

**Who Has Edit Access:** Approver (3 editable fields)

**Entry Conditions:**

- The CC Owner clicks "Submit for Approval" from the Initiated state and all validations pass

**Editable Fields (3):**
Decision (dropdown: Approve / Reject), Risk Level (dropdown: Low / Medium / High), Decision Comments (textarea).

**Available Actions:**

- **Submit Decision:** The Approver sets the Decision field, optionally sets Risk Level and enters Decision Comments, then clicks "Submit Decision." The system reads the Decision field value to determine the next state:
    - Decision = Approve → State transitions to In Implementation
    - Decision = Reject → State transitions to Initiated (loop back)

**Field Display Behaviour:**

- All 25 fields previously filled by the CC Owner are now displayed as read-only (disabled inputs showing the submitted values).
- The 3 Approver fields (Decision, Risk Level, Decision Comments) are displayed as active editable controls.
- Implementation Details fields remain "Not applicable — Available after approval."
- Final Approval fields remain "Not applicable — Pending implementation."
- Implementation Approval Status: "Pending"
- Final Approval Status: "Not Submitted"

**CC Owner's View in This State:**
The CC Owner can view the record but all fields are read-only. The CC Owner does not see the "Submit Decision" button. The CC Owner waits for the Approver's decision.

**Notification Sent on Entry:**
An email notification is sent to the assigned Approver with a task due date of Submission Date + 5 business days.

---

### 3.3.3 In Implementation

**Purpose:** The change has been approved for implementation. The CC Owner carries out the change and documents the implementation details, evidence, and any issues or deviations encountered.

**Who Has Edit Access:** CC Owner (6 editable fields)

**Entry Conditions:**

- The Approver submits a decision of "Approve" from the Pending Implementation Approval state, OR
- A previously submitted record is rejected at the Final Approval gate and returned to this state

**Editable Fields (6):**
Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, Implementation Evidence (file upload).

**Available Actions:**

- **Submit for Final Approval:** The CC Owner clicks "Submit for Final Approval" after completing the implementation detail fields. On success, the state transitions to Pending Final Approval.

**Field Display Behaviour:**

- All original change detail fields (from Initiated state) are displayed as read-only.
- The Approver's decision fields (Decision, Risk Level, Decision Comments) are displayed as read-only showing the approved values.
- The 6 Implementation Details fields are displayed as active editable controls.
- Implementation Approval By and Implementation Approval On are populated with the Approver's name and the approval timestamp (system-generated, read-only).
- Final Approval fields remain "Not applicable — Pending implementation."
- Implementation Approval Status: "Approved"
- Final Approval Status: "Not Submitted"

**Rejection Re-Entry Behaviour:**
When a record returns to In Implementation after rejection at the Final Approval gate, the following applies:

- The 6 Implementation Details fields become editable again for the CC Owner to revise.
- Previously entered implementation values are preserved for revision.
- The previous Final Decision and Final Comments values from the rejected review are overwritten when the Approver re-reviews. The old values are preserved in the audit log.

**Notification Sent on Entry:**
An email notification is sent to the CC Owner with a task due date of Target Closure Date − 3 business days.

---

### 3.3.4 Pending Final Approval

**Purpose:** The CC Owner has completed implementation and submitted evidence. The Approver now reviews the implementation details to confirm the change was executed satisfactorily and the documentation is complete.

**Who Has Edit Access:** Approver (2 editable fields)

**Entry Conditions:**

- The CC Owner clicks "Submit for Final Approval" from the In Implementation state

**Editable Fields (2):**
Final Decision (dropdown: Approve / Reject), Final Comments (textarea).

**Available Actions:**

- **Submit Decision:** The Approver sets the Final Decision field, optionally enters Final Comments, then clicks "Submit Decision." The system reads the Final Decision field value to determine the next state:
    - Final Decision = Approve → State transitions to Closed
    - Final Decision = Reject → State transitions to In Implementation (loop back)

**Field Display Behaviour:**

- All original change detail fields are displayed as read-only.
- The Implementation Approval fields (Decision, Risk Level, Decision Comments) are displayed as read-only showing the previously approved values.
- The 6 Implementation Details fields are displayed as read-only showing the CC Owner's submitted values.
- The 2 Final Approval fields (Final Decision, Final Comments) are displayed as active editable controls.
- Implementation Approval Status: "Approved"
- Final Approval Status: "Pending"

**CC Owner's View in This State:**
The CC Owner can view the record but all fields are read-only. The CC Owner does not see the "Submit Decision" button. The CC Owner waits for the Approver's final decision.

**Notification Sent on Entry:**
An email notification is sent to the assigned Approver with a task due date of Target Closure Date.

---

### 3.3.5 Closed

**Purpose:** The change has been fully approved, implemented, and validated. The record is now a permanent, read-only historical record.

**Who Has Edit Access:** No one (0 editable fields for all roles)

**Entry Conditions:**

- The Approver submits a Final Decision of "Approve" from the Pending Final Approval state

**Editable Fields:** None. All 50 fields are read-only for all roles.

**Available Actions:** None. No workflow action buttons are displayed. Only the "Back to List" navigation link is available.

**Field Display Behaviour:**

- All fields display their final values as read-only.
- All system-generated timestamp and approval-tracking fields are fully populated.
- Final Approval By and Final Approval On are populated with the Approver's name and the approval timestamp.
- Actual Closure Date is system-generated at the moment of closure (the date and time when the Approver's final approval was submitted).
- Implementation Approval Status: "Approved"
- Final Approval Status: "Approved"

**Notification Sent on Entry:**
An email notification is sent to the CC Owner confirming the Change Control has been successfully closed.

**Data Retention:**
Closed records are retained in the system indefinitely. They are never deleted or archived out of the system. They remain accessible to all users through the All Change Controls list and via direct record view.

---

### 3.3.6 Cancelled

**Purpose:** The change request was terminated by the CC Owner before it entered the approval process. The record is preserved as a permanent, read-only record for audit purposes.

**Who Has Edit Access:** No one (0 editable fields for all roles)

**Entry Conditions:**

- The CC Owner clicks "Cancel CC" from the Initiated state and confirms with a mandatory Cancellation Reason via the popup modal

**Editable Fields:** None. All fields are read-only for all roles.

**Available Actions:** None. No workflow action buttons are displayed.

**Field Display Behaviour:**

- All previously entered fields display their values at the time of cancellation as read-only.
- Fields that were "Not applicable" at the Initiated state continue to display as "Not applicable" or "—".
- The Cancellation Reason field, which is hidden in all other states, becomes visible in the Additional Information section below the Comments field. It displays the reason entered during cancellation as a read-only textarea.
- Implementation Approval Status: "N/A"
- Final Approval Status: "N/A"

**Cancellation Restrictions:**

- Cancellation is only possible from the Initiated state. Once a record has been submitted for approval (or is in any subsequent state), it cannot be cancelled.
- Only the CC Owner of that specific record can cancel it. Other users with the CC Owner role cannot cancel someone else's record.
- Cancellation is permanent and cannot be undone. There is no "reopen" or "reactivate" action.

**Notification Sent on Entry:**
If an Approver was previously assigned to the record, an email notification is sent to inform them that the Change Control has been cancelled.

**Data Retention:**
Cancelled records are retained in the system indefinitely with all fields in a read-only state. The Cancellation Reason is permanently visible on the record for audit purposes.

---

### 3.4 State Transition Rules

The following table defines every valid state transition in the system, the triggering action, and who can perform it:

| # | From State | To State | Triggering Action | Performed By | Conditions |
| --- | --- | --- | --- | --- | --- |
| T1 | *(New)* | Initiated | Create CC | CC Owner | User must have CC Owner role |
| T2 | Initiated | Pending Implementation Approval | Submit for Approval | CC Owner (record owner) | All mandatory fields validated; both dates must be in the future; Proposed Implementation Date ≥ 2 business days; Target Closure Date ≥ 10 business days |
| T3 | Initiated | Cancelled | Cancel CC | CC Owner (record owner) | Mandatory Cancellation Reason provided via popup modal |
| T4 | Pending Implementation Approval | In Implementation | Submit Decision | Approver (assigned to record) | Decision field = "Approve" |
| T5 | Pending Implementation Approval | Initiated | Submit Decision | Approver (assigned to record) | Decision field = "Reject" |
| T6 | In Implementation | Pending Final Approval | Submit for Final Approval | CC Owner (record owner) | Implementation detail fields completed |
| T7 | Pending Final Approval | Closed | Submit Decision | Approver (assigned to record) | Final Decision field = "Approve"; system sets Actual Closure Date |
| T8 | Pending Final Approval | In Implementation | Submit Decision | Approver (assigned to record) | Final Decision field = "Reject" |

**Important Notes on Transitions:**

**No Direct State Skipping:** There is no path that skips a state. Every record must pass through the full sequence (or be cancelled from Initiated). A record cannot jump from Initiated directly to In Implementation or from Pending Implementation Approval directly to Closed.

**Decision Field Drives Transition, Not the Button:** At both approval gates, there is a single "Submit Decision" button. The system reads the value of the Decision field (or Final Decision field) to determine whether the transition is an approval or rejection. There are no separate "Approve" and "Reject" buttons.

**Terminal States:** Closed and Cancelled are terminal states. No transitions exit from these states. Once a record reaches either state, it remains there permanently.

**Rejection Loops Are Not Unlimited:** While there is no system-enforced limit on how many times a record can be rejected and resubmitted, each rejection cycle is fully tracked in the audit log. Repeated rejections would be visible in the audit history for management review.

---

### 3.5 Rejection Workflow

Rejection is a critical part of the Change Control lifecycle. It provides a mechanism for the Approver to send a record back for improvement rather than creating a separate "rejected" state. The goal of rejection is to improve the quality of the change documentation, not to terminate the process.

### 3.5.1 Rejection at Implementation Approval Gate (Transition T5)

**Trigger:** The Approver sets the Decision field to "Reject" and clicks "Submit Decision" during the Pending Implementation Approval state.

**What Happens:**

1. The system transitions the record from Pending Implementation Approval back to Initiated.
2. The audit log captures the rejection event, including the Decision value ("Reject"), the Risk Level (if set), and the Decision Comments provided by the Approver.
3. An email notification is sent to the CC Owner informing them that the record has been rejected and needs revision. The notification includes the CC-ID and a summary.
4. The record returns to the Initiated state where the CC Owner has full edit access to all 25 fields.
5. The CC Owner reviews the Approver's feedback (visible in the Decision Comments field, which is now read-only from the CC Owner's perspective in the Initiated state upon return — note: the Approver's previous comments remain visible until the CC Owner resubmits and the Approver overwrites them).
6. The CC Owner revises the necessary fields and resubmits for approval.
7. The record transitions back to Pending Implementation Approval for re-review.
8. The Approver reviews again and sets a new Decision. The new Decision, Risk Level, and Decision Comments overwrite the previous rejection values. The old rejection values are preserved in the audit log.

### 3.5.2 Rejection at Final Approval Gate (Transition T8)

**Trigger:** The Approver sets the Final Decision field to "Reject" and clicks "Submit Decision" during the Pending Final Approval state.

**What Happens:**

1. The system transitions the record from Pending Final Approval back to In Implementation.
2. The audit log captures the rejection event, including the Final Decision value ("Reject") and the Final Comments provided by the Approver.
3. An email notification is sent to the CC Owner informing them that the final review has been rejected and the implementation documentation needs improvement.
4. The record returns to the In Implementation state where the CC Owner has edit access to the 6 implementation detail fields.
5. The CC Owner reviews the Approver's feedback (visible in the Final Comments field).
6. The CC Owner revises the implementation details and resubmits for final approval.
7. The record transitions back to Pending Final Approval for re-review.
8. The Approver reviews again and sets a new Final Decision. The new Final Decision and Final Comments overwrite the previous rejection values. The old rejection values are preserved in the audit log.

### 3.5.3 Rejection Audit Trail Preservation

A critical aspect of the rejection workflow is that **old values are always preserved in the audit log before they are overwritten**. This ensures a complete rejection history is maintained even though the record itself only shows the latest values.

**Example Scenario — Rejection and Re-Approval at Implementation Gate:**

*First Review (Rejection):*

- Audit Entry: Decision changed from [empty] to "Reject"
- Audit Entry: Risk Level changed from [empty] to "Medium"
- Audit Entry: Decision Comments changed from [empty] to "Risk mitigation plan is insufficient. Please provide specific rollback steps for each deployment stage."
- Audit Entry: State changed from "Pending Implementation Approval" to "Initiated"

*Second Review (Approval after revision):*

- Audit Entry: Decision changed from "Reject" to "Approve" — old value "Reject" preserved
- Audit Entry: Risk Level changed from "Medium" to "Low" — old value "Medium" preserved
- Audit Entry: Decision Comments changed from "Risk mitigation plan is insufficient..." to "Revised risk mitigation plan is thorough. Approved for implementation." — old rejection comment preserved

The Change Control record itself now shows Decision = "Approve", Risk Level = "Low", and Decision Comments = "Revised risk mitigation plan is thorough. Approved for implementation." But the audit log retains the full history showing the initial rejection, its rationale, and the subsequent approval.

### 3.5.4 Key Principles of the Rejection Workflow

- **Rejection is not a terminal state.** Rejection sends the record back for improvement. It does not end the process.
- **There is no separate "Rejected" state.** The record returns to its previous state (Initiated or In Implementation) where the CC Owner can make revisions.
- **The same Approver re-reviews.** The assigned Approver does not change after a rejection. The same individual reviews the revised submission.
- **No system-enforced rejection limit.** The system does not cap the number of rejection cycles. However, each cycle is audited and visible for management oversight.
- **Comments are reused, not appended.** The Decision Comments and Final Comments fields are single fields that get overwritten on each review cycle. They are not append-only logs. The audit table serves as the historical record.

---