# Section 4 - Reviewed

---

## 4. SECURITY MATRIX

### 4.1 Overview

The Security Matrix is the definitive reference for field-level permissions in the Change Control module. It defines, for every combination of workflow state and user role, whether each of the 50 fields is **Editable** (the user can modify the value), **Read-Only** (the user can see the value but cannot modify it), or **Not Applicable** (the field is not relevant at this stage and displays a placeholder message instead of a value).

The Security Matrix is maintained as a separate Excel workbook (`Security_Matrix_V1_0.xlsx`) using colour-coded cells:

- **Green cells** indicate the field is editable for that role in that state.
- **Red cells** indicate the field is read-only for that role in that state.

The matrix is structured with the 50 field names across the columns, grouped by their form section, and the state/role combinations down the rows. Each of the six workflow states contains four rows — one for each role (CC Owner, Approver, Viewer, Admin) — resulting in 24 permission rows across 50 field columns.

The Security Matrix governs **field-level permissions only**. Action-level permissions (who can create, submit, cancel, etc.) are documented separately in Section 8.4 of this BRD.

### 4.2 Field-Level Permissions by State & Role

The complete field-level permission matrix is provided in the accompanying Excel workbook. Refer to the authoritative source for the full colour-coded matrix:

**[Insert Security Matrix table here — from Security_Matrix_V1_0.xlsx]**

The following is a summary of the permission distribution across all states and roles:

**Initiated State:**

- CC Owner: 25 fields editable, remaining fields are system-generated (read-only) or not applicable
- Approver: All fields read-only or not applicable
- Viewer: All fields read-only or not applicable
- Admin: All fields read-only or not applicable

**Pending Implementation Approval State:**

- CC Owner: All fields read-only (previously submitted values displayed as disabled inputs)
- Approver: 3 fields editable (Decision, Risk Level, Decision Comments), all others read-only or not applicable
- Viewer: All fields read-only or not applicable
- Admin: All fields read-only or not applicable

**In Implementation State:**

- CC Owner: 6 fields editable (Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, Implementation Evidence), all others read-only
- Approver: All fields read-only
- Viewer: All fields read-only
- Admin: All fields read-only

**Pending Final Approval State:**

- CC Owner: All fields read-only
- Approver: 2 fields editable (Final Decision, Final Comments), all others read-only
- Viewer: All fields read-only
- Admin: All fields read-only

**Closed State:**

- All roles: All 50 fields read-only. No editable fields for any role.

**Cancelled State:**

- All roles: All fields read-only. No editable fields for any role. Cancellation Reason field becomes visible (read-only).

### 4.3 System-Generated vs User-Editable Fields

The 50 fields in the Change Control form fall into two fundamental categories based on how their values are populated.

### 4.3.1 System-Generated Fields (13 fields)

System-generated fields are populated and managed entirely by the system. They are **always read-only** for all users in all states. No user, regardless of role, can directly edit these fields.

| # | Field Name | How It Is Populated |
| --- | --- | --- |
| 1 | CC-ID | Auto-generated unique identifier when a new CC is created (format: CC-XXX) |
| 2 | Current State | Automatically updated when a state transition occurs |
| 3 | Change Owner | Auto-populated with the name of the user who creates the CC record |
| 4 | Last Updated By | Auto-populated with the name of the user who last saved or submitted a change |
| 5 | Created On | Auto-populated with the date and time when the CC record was first created |
| 6 | Last Updated On | Auto-populated with the date and time of the most recent save or submission |
| 7 | Implementation Approval By | Auto-populated with the Approver's name when they submit an Approve decision at the Implementation Approval gate |
| 8 | Implementation Approval On | Auto-populated with the date and time when the Implementation Approval decision was submitted |
| 9 | Final Approval By | Auto-populated with the Approver's name when they submit an Approve decision at the Final Approval gate |
| 10 | Final Approval On | Auto-populated with the date and time when the Final Approval decision was submitted |
| 11 | Implementation Approval Status | System-managed status label that updates based on the workflow state (see Section 4.3.3) |
| 12 | Final Approval Status | System-managed status label that updates based on the workflow state (see Section 4.3.3) |
| 13 | Actual Closure Date | Auto-populated with the date and time when the state transitions to Closed |

### 4.3.2 User-Editable Fields (38 fields)

User-editable fields are populated by users during their designated workflow stages. Which user can edit which field at which state is governed by the Security Matrix. A user-editable field may be editable in one state and read-only in all other states. The 37 user-editable fields are distributed as follows:

- **25 fields** editable by the CC Owner in the Initiated state (Change Details, Planning, Impact & Risk Assessment, Implementation Plan & Validation, Assign Approver, Comments for Approver, Comments)
- **3 fields** editable by the Approver in the Pending Implementation Approval state (Decision, Risk Level, Decision Comments)
- **6 fields** editable by the CC Owner in the In Implementation state (Implementation Details)
- **2 fields** editable by the Approver in the Pending Final Approval state (Final Decision, Final Comments)
- **1 field** editable by the CC Owner during the cancellation action only (Cancellation Reason — entered via popup modal, not directly on the form)
- **25 fields** editable by the CC Owner in the Initiated state
- **3 fields** editable by the Approver in the Pending Implementation Approval state
- **6 fields** editable by the CC Owner in the In Implementation state
- **2 fields** editable by the Approver in the Pending Final Approval state
- **1 field** editable by the CC Owner during the cancellation action only (Cancellation Reason)
- = **37 user-editable fields**
- **13 system-generated fields**
- = **50 total**

Total user-editable + system-generated = 37 + 13 = 50 fields.

### 4.3.3 System-Managed Status Labels

The Implementation Approval Status and Final Approval Status fields are system-managed labels that reflect the current position in the workflow. They are not directly editable by any user. The system sets these values automatically based on the current state:

| Workflow State | Implementation Approval Status | Final Approval Status |
| --- | --- | --- |
| Initiated | Not Submitted | Not Submitted |
| Pending Implementation Approval | Pending | Not Submitted |
| In Implementation | Approved | Not Submitted |
| Pending Final Approval | Approved | Pending |
| Closed | Approved | Approved |
| Cancelled | N/A | N/A |

**Important:** The status label value is "Not Submitted" (not "Not Yet Submitted"). Use the exact values from the table above throughout the system.

### 4.4 Permission Rules

The following rules govern how field-level permissions are applied across the system:

### 4.4.1 Core Permission Principles

**Rule P1 — Single Editor Per State:**
At any given state, at most one role has edit access to any fields. No two roles can edit the same record at the same time. This enforces the shared document model where users take turns.

**Rule P2 — Edit Access Is State-Bound:**
A field that is editable in one state becomes read-only in all subsequent states (with the exception of rejection loops, where the field returns to editable when the record loops back to the previous state).

**Rule P3 — System Fields Are Always Read-Only:**
The 12 system-generated fields (CC-ID, Current State, Change Owner, Last Updated By, Created On, Last Updated On, Implementation Approval By/On, Final Approval By/On, Implementation Approval Status, Final Approval Status) are never editable by any user in any state.

**Rule P4 — Not Applicable Fields Show Placeholder:**
Fields that are not yet relevant to the current workflow stage display a "Not applicable" message with a contextual hint (e.g., "Not applicable — Available after approval"). These fields are not hidden; they are visible but clearly marked as not yet active.

**Rule P5 — Terminal States Lock Everything:**
In the Closed and Cancelled states, all 50 fields are read-only for all roles. No edits are possible.

**Rule P6 — Cancellation Reason Conditional Visibility:**
The Cancellation Reason field (field #50) is hidden in all states except Cancelled. It is only visible in the Cancelled state, displayed as read-only in the Additional Information section below Comments. The value is captured via a popup modal during the cancellation action, not through an inline form field.

**4.4.2 Target Closure Date Permission Rule**

The Target Closure Date field follows the same state-based permission model as all other fields, but its behaviour is worth highlighting:

- **Initiated state:** Editable by CC Owner (part of the 25 editable fields). This applies both on initial creation and when the record returns to Initiated after a rejection at the Implementation Approval gate.
- **All other states (Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, Cancelled):** Read-only. Once the CC Owner submits for approval and the record leaves the Initiated state, the Target Closure Date cannot be modified unless the record is rejected back to Initiated.

### 4.4.3 Rejection and Permission Reset

When a record is rejected and loops back to a previous state, the field permissions reset to match the destination state:

- **Rejection at Implementation Approval (returns to Initiated):** All 25 CC Owner fields become editable again. The Approver's fields (Decision, Risk Level, Decision Comments) retain their values from the rejection but will be overwritten during the next review cycle.
- **Rejection at Final Approval (returns to In Implementation):** The 6 Implementation Details fields become editable for the CC Owner again. The Approver's Final Decision and Final Comments fields retain their values from the rejection but will be overwritten during the next review cycle.

### 4.5 Reference to Security Matrix Excel

The authoritative and complete Security Matrix is maintained in the accompanying Excel workbook:

**File:** `Security_Matrix_V1_0.xlsx`**Sheet:** Change Control
**Version:** 1.0
**Total Fields:** 50
**Total Permission Rows:** 24 (6 states × 4 roles)

This workbook should be treated as a controlled document. Any changes to field permissions must be reflected in both the Security Matrix Excel and this BRD. In the event of a discrepancy between this BRD narrative and the Security Matrix Excel, the **Security Matrix Excel takes precedence** for field-level permission questions.

For action-level permissions (Create CC, Submit for Approval, Cancel CC, etc.), refer to Section 8.4 of this BRD, which is the authoritative source for action permissions.

---