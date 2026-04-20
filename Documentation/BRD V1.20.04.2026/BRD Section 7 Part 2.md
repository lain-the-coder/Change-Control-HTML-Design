# Part 2

---

### 7.5 Implementation Plan & Validation (4 fields)

The Implementation Plan & Validation section captures the CC Owner's planned approach to implementing the change, including how it will be validated and what the fallback plan is if something goes wrong. All four fields are editable by the CC Owner in the Initiated state and read-only in all other states for all roles.

---

### Field 25: High-Level Implementation Plan

**Field ID:** `high_level_implementation_plan`**Section:** Implementation Plan & Validation
**Type:** Textarea (multi-line)
**Mandatory:** Yes
**Max Length:** 2000 characters
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Required — cannot be empty or whitespace only
- Maximum 2000 characters
- Validated at submission time

**Default Value:** None
**Help Text:** "Outline the high-level steps required to implement this change, including the sequence of activities and responsible parties where applicable"
**Display Pattern:** Editable textarea (3 rows) in Initiated state (CC Owner); read-only disabled textarea in all other states
**Example:** "1) Update receipt template UI to render AED symbol using the standard receipt component. 2) Test on QA kiosk device. 3) Deploy to staging. 4) Validate receipt printing. 5) Deploy to production during maintenance window."

---

### Field 26: Validation Approach

**Field ID:** `validation_approach`**Section:** Implementation Plan & Validation
**Type:** Textarea (multi-line)
**Mandatory:** Yes
**Max Length:** 2000 characters
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Required — cannot be empty or whitespace only
- Maximum 2000 characters
- Validated at submission time

**Default Value:** None
**Help Text:** "Describe how the change will be verified or tested to confirm it has been implemented successfully"
**Display Pattern:** Editable textarea (3 rows) in Initiated state (CC Owner); read-only disabled textarea in all other states
**Example:** "Perform smoke testing on payment summary screen (cash/card), verify receipt preview, print a sample receipt, and validate EN/AR locale rendering."

---

### Field 27: Success Criteria

**Field ID:** `success_criteria`**Section:** Implementation Plan & Validation
**Type:** Textarea (multi-line)
**Mandatory:** Yes
**Max Length:** 2000 characters
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Required — cannot be empty or whitespace only
- Maximum 2000 characters
- Validated at submission time

**Default Value:** None
**Help Text:** "Define the measurable criteria that will confirm this change was implemented successfully"
**Display Pattern:** Editable textarea (2 rows) in Initiated state (CC Owner); read-only disabled textarea in all other states
**Example:** "AED symbol and amounts display consistently across payment summary and printed receipts in both EN and AR without layout issues."

---

### Field 28: Rollback / Backout Plan

**Field ID:** `rollback_backout_plan`**Section:** Implementation Plan & Validation
**Type:** Textarea (multi-line)
**Mandatory:** Yes
**Max Length:** 2000 characters
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Required — cannot be empty or whitespace only
- Maximum 2000 characters
- Validated at submission time

**Default Value:** None
**Help Text:** "Describe the actions required to restore the system or process to its previous state in the event of failure"
**Display Pattern:** Editable textarea (3 rows) in Initiated state (CC Owner); read-only disabled textarea in all other states
**Example:** "Revert to previous kiosk package build and redeploy. Restore prior receipt template version from release tag. Confirm payment screens render normally post-rollback."

---

### 7.6 Implementation Details (6 fields)

The Implementation Details section captures the actual outcomes of the implementation — what happened, when, what issues arose, and what evidence was collected. These fields are only editable by the CC Owner in the In Implementation state. In the Initiated and both Pending Approval states, these fields display as "Not applicable — Available after approval." In Closed and Cancelled states, they are read-only.

---

### Field 29: Actual Implementation Date

**Field ID:** `actual_implementation_date`**Section:** Implementation Details
**Type:** Date picker
**Mandatory:** Yes (when submitting for final approval)
**Editable By:** CC Owner in In Implementation state only
**Validation Rules:**

- Required before the CC Owner can submit for final approval
- Should represent the date the change was actually implemented
- No specific minimum lead-time validation (this is a retrospective date, not a future-looking date)

**Default Value:** None
**Help Text:** None
**Display Pattern:**

- Initiated state: "Not applicable — Available after approval"
- Pending Implementation Approval state: "Not applicable — Available after approval"
- In Implementation state: Editable date picker (CC Owner)
- Pending Final Approval state: Read-only disabled date input showing the selected date
- Closed state: Read-only disabled date input
**Example:** "2026-01-28"

---

### Field 30: Post-Implementation Issues

**Field ID:** `post_implementation_issues`**Section:** Implementation Details
**Type:** Dropdown (single select)
**Mandatory:** Yes (when submitting for final approval)
**Editable By:** CC Owner in In Implementation state only
**Dropdown Options:**

- None
- Minor issues resolved
- Issues requiring follow-up

**Validation Rules:**

- Required before the CC Owner can submit for final approval — a value must be selected
- Validated at submission time

**Default Value:** None (placeholder: "Select")
**Help Text:** None
**Display Pattern:**

- Initiated / Pending Implementation Approval: "Not applicable — Available after approval"
- In Implementation: Editable dropdown (CC Owner)
- Pending Final Approval / Closed: Read-only disabled dropdown showing the selected value
**Example:** "Minor issues resolved"

---

### Field 31: Implementation Summary

**Field ID:** `implementation_summary`**Section:** Implementation Details
**Type:** Textarea (multi-line)
**Mandatory:** Yes (when submitting for final approval)
**Max Length:** 2000 characters
**Editable By:** CC Owner in In Implementation state only
**Validation Rules:**

- Required before submitting for final approval — cannot be empty or whitespace only
- Maximum 2000 characters

**Default Value:** None
**Help Text:** None
**Display Pattern:**

- Initiated / Pending Implementation Approval: "Not applicable — Available after approval"
- In Implementation: Editable textarea (CC Owner)
- Pending Final Approval / Closed: Read-only disabled textarea
**Example:** "Receipt template updated to use AED symbol from standard currency component. Deployed to QA, tested on kiosk device, validated print output. Deployed to production on 28 Jan during 02:00–03:30 maintenance window. No service interruption."

---

### Field 32: Deviations from Plan

**Field ID:** `deviations_from_plan`**Section:** Implementation Details
**Type:** Textarea (multi-line)
**Mandatory:** No
**Max Length:** 2000 characters
**Editable By:** CC Owner in In Implementation state only
**Validation Rules:**

- Optional field
- Maximum 2000 characters

**Default Value:** None
**Help Text:** None
**Display Pattern:**

- Initiated / Pending Implementation Approval: "Not applicable — Available after approval"
- In Implementation: Editable textarea (CC Owner)
- Pending Final Approval / Closed: Read-only disabled textarea
**Example:** "Implementation window extended by 30 minutes due to unexpected cache invalidation step. No impact on service availability."

---

### Field 33: Validation Performed

**Field ID:** `validation_performed`**Section:** Implementation Details
**Type:** Textarea (multi-line)
**Mandatory:** Yes (when submitting for final approval)
**Max Length:** 2000 characters
**Editable By:** CC Owner in In Implementation state only
**Validation Rules:**

- Required before submitting for final approval — cannot be empty or whitespace only
- Maximum 2000 characters

**Default Value:** None
**Help Text:** None
**Display Pattern:**

- Initiated / Pending Implementation Approval: "Not applicable — Available after approval"
- In Implementation: Editable textarea (CC Owner)
- Pending Final Approval / Closed: Read-only disabled textarea
**Example:** "Smoke testing completed for cash and card payments. Receipt preview and printed output validated in EN and AR."

---

### Field 34: Implementation Evidence

**Field ID:** `implementation_evidence`**Section:** Implementation Details
**Type:** File upload
**Mandatory:** Yes (when submitting for final approval)
**Editable By:** CC Owner in In Implementation state only
**Validation Rules:**

- Required before the CC Owner can submit for final approval
- Supported file types: PDF, DOCX, XLSX, PNG, JPG
- Maximum file size: 10MB per file
- Single file upload only. Users should combine related evidence into one file before uploading.

**Default Value:** None (no files uploaded)
**Help Text:** "PDF, DOCX, XLSX, PNG, JPG (Max 10MB)"
**Display Pattern:**

- Initiated / Pending Implementation Approval: Upload box displayed as disabled with "Not applicable — Available after approval" message
- In Implementation: Active upload box (CC Owner)
- Pending Final Approval / Closed: Read-only display of uploaded file names (e.g., "UAT_Scripts.pdf (uploaded)")
**Example:** "UAT_Scripts.pdf"

---

### 7.7 Approvals — Initiation (2 fields)

The Approvals — Initiation section captures the CC Owner's approver selection and any comments they want the Approver to consider during review. Both fields are editable by the CC Owner in the Initiated state and read-only in all other states.

---

### Field 35: Assign Approver

**Field ID:** `assign_approver`**Section:** Approvals — Initiation
**Type:** Dropdown (single select, dynamic — populated from user database)
**Mandatory:** Yes
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Required — an Approver must be selected before submission
- The dropdown shall only display users who currently hold the Approver role in the system. Users with CC Owner, Viewer, or Admin roles shall not appear.
- Since each user holds only one role at a time, the CC Owner (who holds the CC Owner role) will never appear in this dropdown.
- The system prevents the edge case where a CC Owner's role could be changed to Approver while they have active records: Admin cannot change a user's role if they are associated with any active CC records (see FR-6.2.29 and BR-8.4.11). This structurally prevents a CC Owner from ever becoming selectable as Approver on their own record.
- Changes to this field are tracked in the audit log.

**Default Value:** None (placeholder: "Select Approver")
**Help Text:** "Select the person who will approve this change"
**Display Pattern:** Editable dropdown in Initiated state (CC Owner); read-only disabled dropdown showing the selected Approver's name in all other states
**Example:** "Jane Smith"

---

### Field 36: Comments for Approver

**Field ID:** `comments_for_approver`**Section:** Approvals — Initiation
**Type:** Textarea (multi-line)
**Mandatory:** No
**Max Length:** 2000 characters
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Optional field
- Maximum 2000 characters

**Default Value:** None
**Help Text:** "Optional comments for the approver"
**Display Pattern:** Editable textarea (3 rows) in Initiated state (CC Owner); read-only disabled textarea in all other states
**Example:** "Please review for implementation approval. Change is UI-only; testing planned in QA before rollout."

---

### 7.8 Approvals — Implementation Approval (5 fields)

The Implementation Approval section contains the Approver's decision fields and the system-generated approval tracking fields. Three fields (Decision, Risk Level, Decision Comments) are editable by the Approver in the Pending Implementation Approval state. Two fields (Implementation Approval By, Implementation Approval On) are system-generated and always read-only.

---

### Field 37: Decision

**Field ID:** `decision`**Section:** Approvals — Implementation Approval
**Type:** Dropdown (single select)
**Mandatory:** Yes (when Approver submits decision)
**Editable By:** Approver in Pending Implementation Approval state only
**Dropdown Options:**

- Approve
- Reject

**Validation Rules:**

- Required when the Approver clicks "Submit Decision"
- The value of this field determines the state transition (see Section 3.4, Transitions T4 and T5):
    - "Approve" → state transitions to In Implementation
    - "Reject" → state transitions to Initiated (loop back)
- This field is overwritten if the record is rejected and later re-reviewed. The old value is preserved in the audit log before overwrite.
- Changes to this field are always tracked in the audit log.

**Default Value:** None (placeholder: "Select" or no selection)
**Help Text:** None
**Display Pattern:**

- Initiated state: "Not applicable — Pending submission"
- Pending Implementation Approval state: Editable dropdown (Approver only)
- In Implementation / Pending Final Approval / Closed: Read-only disabled dropdown showing the decision value (e.g., "Approve")
- Cancelled: "N/A"
**Example:** "Approve"

---

### Field 38: Risk Level

**Field ID:** `risk_level`**Section:** Approvals — Implementation Approval
**Type:** Dropdown (single select)
**Mandatory:** Yes (when Approver submits decision)
**Editable By:** Approver in Pending Implementation Approval state only
**Dropdown Options:**

- Low
- Medium
- High

**Important:** Risk Level is set exclusively by the Approver, not by the CC Owner. The CC Owner provides their own risk rationale in the Impact & Risk Assessment section (field 22: Risk Rationale), but the formal Risk Level classification is an independent Approver assessment.

**Validation Rules:**

- Required when the Approver clicks "Submit Decision" — submission is blocked if Risk Level is not selected
- This field is overwritten if the record is rejected and later re-reviewed. The old value is preserved in the audit log before overwrite.
- Changes to this field are always tracked in the audit log.

**Default Value:** None (placeholder: "Select" or no selection)
**Help Text:** "Not applicable — Will be set by approver during review" (shown to CC Owner in Initiated state)
**Display Pattern:**

- Initiated state: "Not applicable — Will be set by approver during review"
- Pending Implementation Approval state: Editable dropdown (Approver only)
- In Implementation / Pending Final Approval / Closed: Read-only disabled dropdown showing the selected risk level
- Cancelled: "N/A"
**Example:** "Low"

---

### Field 39: Decision Comments

**Field ID:** `decision_comments`**Section:** Approvals — Implementation Approval
**Type:** Textarea (multi-line)
**Mandatory:**  Yes (when Approver submits decision)
**Max Length:** 2000 characters
**Editable By:** Approver in Pending Implementation Approval state only
**Validation Rules:**

- Required when the Approver clicks "Submit Decision" — submission is blocked if Decision Comments is empty or whitespace only
- Maximum 2000 characters
- Used for both Approve and Reject decisions — there is no separate "Rejection Comments" field
- This field is overwritten if the record is rejected and later re-reviewed. The old value (including rejection rationale) is preserved in the audit log before overwrite.
- Changes to this field are always tracked in the audit log.

**Default Value:** None
**Help Text:** "Provide rationale for your decision"
**Display Pattern:**

- Initiated state: "Not applicable — Pending submission"
- Pending Implementation Approval state: Editable textarea (Approver only)
- In Implementation / Pending Final Approval / Closed: Read-only disabled textarea showing the Approver's comments
- Cancelled: "N/A"
**Example (Approval):** "All requirements met. Risk is low — UI-only change with adequate testing plan and rollback procedure."
**Example (Rejection):** "Risk mitigation plan is insufficient. Please provide specific rollback steps for each deployment stage."

---

### Field 40: Implementation Approval By

**Field ID:** `implementation_approval_by`**Section:** Approvals — Implementation Approval
**Type:** System-generated text
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-generated when the Approver submits an Approve decision
**Validation Rules:**

- Auto-populated with the Approver's full name when Decision = "Approve" is submitted
- Only populated on approval, not on rejection
- Read-only for all roles in all states

**Default Value:** "—" (dash, indicating not yet populated)
**Help Text:** None
**Display Pattern:** System-managed read-only value. Displays "—" until the Implementation Approval decision is submitted as Approve, then displays the Approver's name.
**Example:** "Jane Smith"

---

### Field 41: Implementation Approval On

**Field ID:** `implementation_approval_on`**Section:** Approvals — Implementation Approval
**Type:** System-generated datetime
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-generated when the Approver submits an Approve decision
**Validation Rules:**

- Auto-populated with the current date and time when Decision = "Approve" is submitted
- Only populated on approval, not on rejection
- Read-only for all roles in all states

**Default Value:** "—" (dash, indicating not yet populated)
**Help Text:** None
**Display Pattern:** System-managed read-only value. Displays "—" until approval, then displays the timestamp in the same format as Created On (e.g., "26 Jan 2026, 4:00 PM").
**Example:** "26 Jan 2026, 4:00 PM"