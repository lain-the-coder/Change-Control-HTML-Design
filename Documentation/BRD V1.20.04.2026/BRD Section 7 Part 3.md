# Part 3

---

### 7.9 Approvals — Final Approval (4 fields)

The Final Approval section contains the Approver's final decision fields and the system-generated final approval tracking fields. Two fields (Final Decision, Final Comments) are editable by the Approver in the Pending Final Approval state. Two fields (Final Approval By, Final Approval On) are system-generated and always read-only.

---

### Field 42: Final Decision

**Field ID:** `final_decision`**Section:** Approvals — Final Approval
**Type:** Dropdown (single select)
**Mandatory:** Yes (when Approver submits final decision)
**Editable By:** Approver in Pending Final Approval state only
**Dropdown Options:**

- Approve
- Reject

**Validation Rules:**

- Required when the Approver clicks "Submit Decision" at the Final Approval gate
- The value of this field determines the state transition (see Section 3.4, Transitions T7 and T8):
    - "Approve" → state transitions to Closed
    - "Reject" → state transitions to In Implementation (loop back)
- This field is overwritten if the record is rejected and later re-reviewed. The old value is preserved in the audit log before overwrite.
- Changes to this field are always tracked in the audit log.

**Default Value:** None (placeholder: "Select" or no selection)
**Help Text:** None
**Display Pattern:**

- Initiated / Pending Implementation Approval / In Implementation: "Not applicable — Pending implementation"
- Pending Final Approval: Editable dropdown (Approver only)
- Closed: Read-only disabled dropdown showing the decision value (e.g., "Approve")
- Cancelled: "N/A"
**Example:** "Approve"

---

### Field 43: Final Comments

**Field ID:** `final_comments`**Section:** Approvals — Final Approval
**Type:** Textarea (multi-line)
**Mandatory:** Yes (when Approver submits final decision)
**Max Length:** 2000 characters
**Editable By:** Approver in Pending Final Approval state only
**Validation Rules:**

- Required when the Approver clicks "Submit Decision" at the Final Approval gate — submission is blocked if Final Comments is empty or whitespace only
- Maximum 2000 characters
- Used for both Approve and Reject final decisions — there is no separate "Final Rejection Comments" field
- This field is overwritten if the record is rejected and later re-reviewed. The old value (including rejection rationale) is preserved in the audit log before overwrite.
- Changes to this field are always tracked in the audit log.

**Default Value:** None
**Help Text:** None
**Display Pattern:**

- Initiated / Pending Implementation Approval / In Implementation: "Not applicable — Pending implementation"
- Pending Final Approval: Editable textarea (Approver only)
- Closed: Read-only disabled textarea showing the Approver's final comments
- Cancelled: "N/A"
**Example (Approval):** "Implementation completed as planned. Evidence reviewed and satisfactory. Change approved for closure."
**Example (Rejection):** "Validation evidence is incomplete. Please provide test results for the Arabic locale receipt output."

---

### Field 44: Final Approval By

**Field ID:** `final_approval_by`**Section:** Approvals — Final Approval
**Type:** System-generated text
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-generated when the Approver submits a Final Decision of Approve
**Validation Rules:**

- Auto-populated with the Approver's full name when Final Decision = "Approve" is submitted
- Only populated on approval, not on rejection
- Read-only for all roles in all states

**Default Value:** "—" (dash, indicating not yet populated)
**Help Text:** None
**Display Pattern:** System-managed read-only value. Displays "—" until the Final Approval decision is submitted as Approve, then displays the Approver's name.
**Example:** "Jane Smith"

---

### Field 45: Final Approval On

**Field ID:** `final_approval_on`**Section:** Approvals — Final Approval
**Type:** System-generated datetime
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-generated when the Approver submits a Final Decision of Approve
**Validation Rules:**

- Auto-populated with the current date and time when Final Decision = "Approve" is submitted
- Only populated on approval, not on rejection
- Read-only for all roles in all states

**Default Value:** "—" (dash, indicating not yet populated)
**Help Text:** None
**Display Pattern:** System-managed read-only value. Displays "—" until final approval, then displays the timestamp in the standard format (e.g., "29 Jan 2026, 12:00 PM").
**Example:** "29 Jan 2026, 12:00 PM"

---

### 7.10 Approvals — Status (3 fields)

The Status section contains system-managed status labels that indicate the current position in the approval lifecycle, plus the Actual Closure Date which is set when the record reaches the Closed state. All three fields are system-generated and read-only for all users in all states.

---

### Field 46: Implementation Approval Status

**Field ID:** `implementation_approval_status`**Section:** Approvals — Status
**Type:** System-managed text (status label)
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-managed, updated automatically based on the current workflow state
**Valid Values:**

- Not Submitted
- Pending
- Approved
- N/A

**Validation Rules:**

- The system sets this value automatically based on the current state. It cannot be directly edited by any user.
- Value mapping by state:

| Workflow State | Implementation Approval Status |
| --- | --- |
| Initiated | Not Submitted |
| Pending Implementation Approval | Pending |
| In Implementation | Approved |
| Pending Final Approval | Approved |
| Closed | Approved |
| Cancelled | N/A |

**Important:** The value is "Not Submitted" — not "Not Yet Submitted." Use the exact value as specified.

**Default Value:** "Not Submitted" (set on creation)
**Help Text:** None
**Display Pattern:** System-managed read-only value (meta-value style), displayed in the Status subsection of the Approvals card.
**Example:** "Pending"

---

### Field 47: Final Approval Status

**Field ID:** `final_approval_status`**Section:** Approvals — Status
**Type:** System-managed text (status label)
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-managed, updated automatically based on the current workflow state
**Valid Values:**

- Not Submitted
- Pending
- Approved
- N/A

**Validation Rules:**

- The system sets this value automatically based on the current state. It cannot be directly edited by any user.
- Value mapping by state:

| Workflow State | Final Approval Status |
| --- | --- |
| Initiated | Not Submitted |
| Pending Implementation Approval | Not Submitted |
| In Implementation | Not Submitted |
| Pending Final Approval | Pending |
| Closed | Approved |
| Cancelled | N/A |

**Important:** The value is "Not Submitted" — not "Not Yet Submitted." Use the exact value as specified.

**Default Value:** "Not Submitted" (set on creation)
**Help Text:** None
**Display Pattern:** System-managed read-only value (meta-value style), displayed alongside Implementation Approval Status in a 2-column grid.
**Example:** "Approved"

---

### Field 48: Actual Closure Date

**Field ID:** `actual_closure_date`**Section:** Approvals — Status
**Type:** System-generated datetime
**Mandatory:** Automatic (system-managed)
**Editable By:** No user — system-generated when the state transitions to Closed
**Validation Rules:**

- Auto-populated with the current date and time at the moment the Approver submits Final Decision = "Approve" and the state transitions to Closed
- Only populated when the record reaches the Closed state — remains empty ("—") in all other states
- Read-only for all roles in all states
- Never populated for Cancelled records

**Default Value:** "—" (dash, indicating not yet populated)
**Help Text:** "System-captured when CC is closed"
**Display Pattern:** System-managed read-only value. Displays "—" in all states until the record is closed, then displays the closure timestamp in the standard format.
**Example:** "29 Jan 2026, 12:00 PM"

---

### 7.11 Additional Information (2 fields)

The Additional Information section contains the general Comments field and the Cancellation Reason field. Comments is editable by the CC Owner in the Initiated state. Cancellation Reason is a special field that is hidden in all states except Cancelled, and its value is captured via a popup modal during the cancellation action rather than through an inline form control.

---

### Field 49: Comments

**Field ID:** `comments`**Section:** Additional Information
**Type:** Textarea (multi-line)
**Mandatory:** No
**Max Length:** 2000 characters
**Editable By:** CC Owner in Initiated state only
**Validation Rules:**

- Optional field
- Maximum 2000 characters

**Default Value:** None
**Help Text:** "Add any additional information or context"
**Display Pattern:** Editable textarea (4 rows) in Initiated state (CC Owner); read-only disabled textarea in all other states. Always visible in all states (unlike Cancellation Reason which is conditionally visible).
**Example:** "This change aligns with our Q2 digital transformation initiative. No backend changes. No DB changes. UI-only update with kiosk validation planned."

---

### Field 50: Cancellation Reason

**Field ID:** `cancellation_reason`**Section:** Additional Information
**Type:** Textarea (multi-line)
**Mandatory:** Yes (when cancelling a CC)
**Max Length:** 500 characters
**Editable By:** CC Owner during the cancellation action only (entered via popup modal, not through an inline form field)

**Validation Rules:**

- Required when cancelling — cannot be empty or whitespace only
- Maximum 500 characters
- Validated within the cancellation popup modal before confirmation is allowed
- This field is not editable through the normal form interface. The value is captured exclusively through the cancellation modal and saved to the record on confirmation.
- Once saved, the value is permanently read-only and cannot be modified.

**Default Value:** None (field is hidden until cancellation occurs)
**Help Text:** "Provide reason for cancelling this Change Control (required)" (shown in the cancellation popup modal)

**Visibility Rules:**

- **Initiated state:** Hidden — the field does not appear on the form. The value is entered through the cancellation popup modal.
- **Pending Implementation Approval state:** Hidden
- **In Implementation state:** Hidden
- **Pending Final Approval state:** Hidden
- **Closed state:** Hidden
- **Cancelled state:** Visible — displayed in the Additional Information section below the Comments field as a read-only textarea showing the reason that was entered during cancellation.

**Display Pattern:** Conditional visibility (the 5th field display pattern — see Section 9.2.5). When visible in the Cancelled state, displayed as a read-only disabled textarea.
**Example:** "Business requirements changed after stakeholder review, this change is no longer needed."

---

### 7.12 Field Summary

The following table provides a complete summary of all 50 fields for quick reference. For detailed definitions, validation rules, and display patterns, refer to the individual field definitions in Sections 7.1 through 7.11.

| # | Field Name | Section | Type | Mandatory | Editable By |
| --- | --- | --- | --- | --- | --- |
| 1 | CC-ID | Identification | System-generated | Auto | System |
| 2 | Current State | Identification | System-managed | Auto | System |
| 3 | Change Owner | Identification | System-generated | Auto | System |
| 4 | Last Updated By | Identification | System-generated | Auto | System |
| 5 | Created On | Identification | System-generated | Auto | System |
| 6 | Last Updated On | Identification | System-generated | Auto | System |
| 7 | Change Title | Change Definition | Text input | Yes | CC Owner — Initiated |
| 8 | Change Description | Change Definition | Textarea | Yes | CC Owner — Initiated |
| 9 | Change Type | Change Definition | Dropdown | Yes | CC Owner — Initiated |
| 10 | Change Category | Change Definition | Dropdown | Yes | CC Owner — Initiated |
| 11 | Department / Function | Change Definition | Dropdown | Yes | CC Owner — Initiated |
| 12 | Affected Systems / Modules | Change Definition | Text input | Yes | CC Owner — Initiated |
| 13 | Proposed Implementation Date | Planning | Date picker | Yes | CC Owner — Initiated |
| 14 | Target Closure Date | Planning | Date picker | Yes | CC Owner — Initiated |
| 15 | Implementation Window Start | Planning | Time picker | No | CC Owner — Initiated |
| 16 | Implementation Window End | Planning | Time picker | No | CC Owner — Initiated |
| 17 | Reason for Change | Impact & Risk | Textarea | Yes | CC Owner — Initiated |
| 18 | Business Impact | Impact & Risk | Textarea | Yes | CC Owner — Initiated |
| 19 | Expected Downtime | Impact & Risk | Dropdown | Yes | CC Owner — Initiated |
| 20 | Requires Testing | Impact & Risk | Dropdown | Yes | CC Owner — Initiated |
| 21 | Requires Training | Impact & Risk | Dropdown | Yes | CC Owner — Initiated |
| 22 | Risk Rationale | Impact & Risk | Textarea | Yes | CC Owner — Initiated |
| 23 | Key Risks & Mitigations | Impact & Risk | Textarea | Yes | CC Owner — Initiated |
| 24 | Supporting Documents | Impact & Risk | File upload (single) | No | CC Owner — Initiated |
| 25 | High-Level Implementation Plan | Impl Plan & Validation | Textarea | Yes | CC Owner — Initiated |
| 26 | Validation Approach | Impl Plan & Validation | Textarea | Yes | CC Owner — Initiated |
| 27 | Success Criteria | Impl Plan & Validation | Textarea | Yes | CC Owner — Initiated |
| 28 | Rollback / Backout Plan | Impl Plan & Validation | Textarea | Yes | CC Owner — Initiated |
| 29 | Actual Implementation Date | Implementation Details | Date picker | Yes* | CC Owner — In Implementation |
| 30 | Post-Implementation Issues | Implementation Details | Textarea | Yes* | CC Owner — In Implementation |
| 31 | Implementation Summary | Implementation Details | Textarea | Yes* | CC Owner — In Implementation |
| 32 | Deviations from Plan | Implementation Details | Textarea | No | CC Owner — In Implementation |
| 33 | Validation Performed | Implementation Details | Textarea | Yes* | CC Owner — In Implementation |
| 34 | Implementation Evidence | Implementation Details | File upload (single) | Yes* | CC Owner — In Implementation |
| 35 | Assign Approver | Approvals — Initiation | Dropdown (dynamic) | Yes | CC Owner — Initiated |
| 36 | Comments for Approver | Approvals — Initiation | Textarea | No | CC Owner — Initiated |
| 37 | Decision | Approvals — Impl Approval | Dropdown | Yes** | Approver — Pending Impl Approval |
| 38 | Risk Level | Approvals — Impl Approval | Dropdown | Yes** | Approver — Pending Impl Approval |
| 39 | Decision Comments | Approvals — Impl Approval | Textarea | Yes** | Approver — Pending Impl Approval |
| 40 | Implementation Approval By | Approvals — Impl Approval | System-generated | Auto | System |
| 41 | Implementation Approval On | Approvals — Impl Approval | System-generated | Auto | System |
| 42 | Final Decision | Approvals — Final Approval | Dropdown | Yes** | Approver — Pending Final Approval |
| 43 | Final Comments | Approvals — Final Approval | Textarea | Yes** | Approver — Pending Final Approval |
| 44 | Final Approval By | Approvals — Final Approval | System-generated | Auto | System |
| 45 | Final Approval On | Approvals — Final Approval | System-generated | Auto | System |
| 46 | Implementation Approval Status | Approvals — Status | System-managed | Auto | System |
| 47 | Final Approval Status | Approvals — Status | System-managed | Auto | System |
| 48 | Actual Closure Date | Approvals — Status | System-generated | Auto | System |
| 49 | Comments | Additional Information | Textarea | No | CC Owner — Initiated |
| 50 | Cancellation Reason | Additional Information | Textarea | Yes*** | CC Owner — Cancellation action only |

**Table Notes:**

- Mandatory when submitting for final approval (from In Implementation state)
- * Mandatory when submitting decision at the respective approval gate
- ** Mandatory only when cancelling a CC (entered via popup modal)
- "System" in the Editable By column means the field is system-generated and read-only for all users in all states
- All fields are read-only for all roles in the Closed and Cancelled states

**Field Count Verification:**

- System-generated fields: 13 (fields 1–6, 40–41, 44–45, 46–48)
- User-editable fields: 37 (fields 7–39, 42–43, 49–50)
- **Total: 50 fields** ✓