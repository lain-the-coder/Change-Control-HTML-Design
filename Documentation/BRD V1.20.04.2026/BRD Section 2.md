# Section 2 - Reviewed

---

## 2. USER ROLES & PERSONAS

### 2.1 Role Definitions

The Change Control module defines four distinct user roles. Every user in the system is assigned exactly one role by an Admin. A user's role determines what actions they can perform, which fields they can edit, and at which workflow states they have write access.

### 2.1.1 CC Owner

**Role Purpose:** The CC Owner is the person who creates, prepares, and drives a Change Control through its full lifecycle. They are responsible for documenting the change request, completing the implementation, and providing evidence of completion.

**Typical Persona:** A process owner, team lead, project engineer, IT specialist, or operations manager who identifies the need for a change and takes responsibility for executing it.

**System Behaviour:**

- When a user with the CC Owner role creates a new Change Control, the system automatically populates the "Change Owner" field with that user's name. This field is system-generated and cannot be changed.
- The CC Owner is the owner of that specific record only. Other users who also hold the CC Owner role in the system are not owners of that record and cannot perform owner-specific actions (such as cancelling) on it.
- A user with the CC Owner role can create multiple Change Controls. Each record is independently owned by the user who created it.

**Key Capabilities:**

- Create new Change Control records
- Edit 25 fields during the Initiated state (change details, planning, impact assessment, implementation plan, approver assignment, and comments)
- Submit the record for implementation approval
- Cancel the record (only from the Initiated state, only for records they own)
- Edit 6 implementation detail fields during the In Implementation state
- Submit the record for final approval after implementation is complete
- View all Change Controls in the system (regardless of ownership)

---

### 2.1.2 Approver

**Role Purpose:** The Approver is the independent reviewer responsible for evaluating the change request at two approval gates. They assess the change for completeness, risk, and readiness — first before implementation begins, and again after implementation is completed.

**Typical Persona:** A quality manager, department head, compliance officer, or senior technical authority who has the knowledge and authority to evaluate proposed changes and their associated risks.

**System Behaviour:**

- An Approver does not self-assign to a record. The CC Owner selects the Approver from a dropdown during the Initiated state. The dropdown only displays users who hold the Approver role.
- Once assigned, the same Approver reviews the record at both approval gates (Implementation Approval and Final Approval).
- The Approver receives email notifications when a record is submitted for their review, including a task due date.

**Key Capabilities:**

- Edit 3 fields during the Pending Implementation Approval state (Decision, Risk Level, Decision Comments)
- Edit 2 fields during the Pending Final Approval state (Final Decision, Final Comments)
- Submit their decision at each approval gate using the "Submit Decision" button
- Approve or reject at each gate by setting the Decision field value before submitting
- View all Change Controls in the system
- Access the Approvals queue showing records pending their review

**Key Restrictions:**

- Cannot create Change Controls
- Cannot cancel Change Controls
- Cannot edit any fields outside of their designated approval states
- Cannot approve a record or even be assigned as an Approver where they are also the CC Owner (segregation of duties)

---

### 2.1.3 Viewer

**Role Purpose:** The Viewer has read-only access to all Change Controls in the system. This role exists for stakeholders, auditors, or team members who need visibility into the change management process but do not participate in creating, approving, or implementing changes.

**Typical Persona:** A compliance auditor, executive sponsor, project stakeholder, or team member who needs to monitor change activity without directly participating in the workflow.

**System Behaviour:**

- Viewers can access and read all Change Control records across all workflow states.
- All fields appear as read-only or system-managed to the Viewer at every state.
- Viewers do not receive workflow-related email notifications (they are not participants in the approval or implementation process).

**Key Capabilities:**

- View all Change Controls in the system
- View all fields and sections of any Change Control record
- Access the Dashboard overview statistics
- Access the All Change Controls list view

**Key Restrictions:**

- Cannot create, edit, submit, approve, reject, or cancel any Change Control
- Zero editable fields in any state
- No workflow action buttons are displayed to this role

---

### 2.1.4 Admin

**Role Purpose:** The Admin manages system configuration and user accounts. The Admin role is focused on platform administration rather than Change Control workflow participation.

**Typical Persona:** A system administrator, IT administrator, or QMS platform manager responsible for maintaining user accounts and system settings.

**System Behaviour:**

- Admins have view-only access to all Change Control records, identical to the Viewer role from a CC perspective.
- Admins have exclusive access to the user management functions within the Settings area.
- Admins can create users, assign roles, edit user profiles, and deactivate users.

**Key Capabilities:**

- View all Change Controls in the system (read-only, same as Viewer)
- Create new user accounts (Full Name, Email, Password, Role)
- Edit existing user profiles: Full Name and Role only (Email is set at creation and cannot be changed; password resets are handled through the Forgot Password flow or at the database level)
- Deactivate user accounts
- Access the full Settings interface (Profile and User Management tabs)

**Key Restrictions:**

- Cannot create, edit, submit, approve, reject, or cancel any Change Control
- Zero editable fields on CC records in any state
- No workflow action buttons are displayed to this role on CC records
- Cannot change a user's role if that user has any active CC records (records in any state other than Closed or Cancelled) — see Section 2.4 for details
- Admin user management actions are captured in the audit log

---

### 2.2 Role Responsibilities

The following table summarizes each role's responsibilities across the Change Control lifecycle:

| Lifecycle Phase | CC Owner | Approver | Viewer | Admin |
| --- | --- | --- | --- | --- |
| **Record Creation** | Creates the CC record; system auto-populates Change Owner | No involvement | No involvement | No involvement |
| **Change Documentation (Initiated)** | Fills all 25 editable fields: change details, planning dates, impact and risk assessment, implementation plan, approver assignment, and comments | No involvement | Can view the record | Can view the record |
| **Submission for Approval** | Clicks "Submit for Approval" after all mandatory fields pass validation | No involvement | No involvement | No involvement |
| **Implementation Approval Review** | Waits; can view the record in read-only mode | Reviews the change; sets Decision (Approve/Reject), Risk Level, and Decision Comments; clicks "Submit Decision" | Can view the record | Can view the record |
| **Rejection at Implementation Approval** | Receives rejection notification; revises and resubmits | Provides rejection rationale in Decision Comments | Can view the record | Can view the record |
| **Implementation (In Implementation)** | Completes the 6 implementation detail fields: Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, Implementation Evidence | No involvement | Can view the record | Can view the record |
| **Submission for Final Approval** | Clicks "Submit for Final Approval" | No involvement | No involvement | No involvement |
| **Final Approval Review** | Waits; can view the record in read-only mode | Reviews implementation evidence; sets Final Decision (Approve/Reject) and Final Comments; clicks "Submit Decision" | Can view the record | Can view the record |
| **Rejection at Final Approval** | Receives rejection notification; improves implementation documentation and resubmits | Provides rejection rationale in Final Comments | Can view the record | Can view the record |
| **Closure** | Receives success notification; record becomes read-only | No further action; record becomes read-only | Can view the record | Can view the record |
| **Cancellation** | Can cancel only from Initiated state; must provide mandatory Cancellation Reason via popup modal | Receives notification if previously assigned; cannot cancel | Can view the record | Can view the record |
| **User Management** | No access | No access | No access |  Creates users (Full Name, Email, Password, Role); edits existing users (Full Name and Role only); deactivates users; cannot change role for users with active CC records |

---

### 2.3 Role-Based Access Summary

### 2.3.1 Editable Field Counts by State and Role

The Security Matrix defines exactly which fields each role can edit at each workflow state. The following table summarises the editable field counts:

| Workflow State | CC Owner | Approver | Viewer | Admin |
| --- | --- | --- | --- | --- |
| **Initiated** | 25 editable fields | 0 (read-only) | 0 (read-only) | 0 (read-only) |
| **Pending Implementation Approval** | 0 (read-only) | 3 editable fields | 0 (read-only) | 0 (read-only) |
| **In Implementation** | 6 editable fields | 0 (read-only) | 0 (read-only) | 0 (read-only) |
| **Pending Final Approval** | 0 (read-only) | 2 editable fields | 0 (read-only) | 0 (read-only) |
| **Closed** | 0 (read-only) | 0 (read-only) | 0 (read-only) | 0 (read-only) |
| **Cancelled** | 0 (read-only) | 0 (read-only) | 0 (read-only) | 0 (read-only) |

**Key Principle:** At any given state, only ONE role has edit access to specific fields. No two roles can edit the same record simultaneously. This enforces the shared document model where users take turns editing their designated fields during their designated stages.

### 2.3.2 Editable Fields by State — Detailed Breakdown

**Initiated State — CC Owner (25 fields):**
Change Title, Change Description, Change Type, Change Category, Department/Function, Affected Systems/Modules, Proposed Implementation Date, Target Closure Date, Implementation Window Start, Implementation Window End, Reason for Change, Business Impact, Expected Downtime, Requires Testing, Requires Training, Risk Rationale, Key Risks & Mitigations, Supporting Documents, High-Level Implementation Plan, Validation Approach, Success Criteria, Rollback/Backout Plan, Assign Approver, Comments for Approver, Comments.

**Pending Implementation Approval State — Approver (3 fields):**
Decision, Risk Level, Decision Comments.

**In Implementation State — CC Owner (6 fields):**
Actual Implementation Date, Post-Implementation Issues, Implementation Summary, Deviations from Plan, Validation Performed, Implementation Evidence.

**Pending Final Approval State — Approver (2 fields):**
Final Decision, Final Comments.

### 2.3.3 Action Permissions by Role

| Action | CC Owner | Approver | Viewer | Admin |
| --- | --- | --- | --- | --- |
| **Create CC** | Yes (always) | No | No | No |
| **Submit for Approval** | Yes (from Initiated, own record only) | No | No | No |
| **Cancel CC** | Yes (from Initiated, own record only) | No | No | No |
| **Submit Decision (Implementation)** | No | Yes (from Pending Implementation Approval, assigned record only) | No | No |
| **Submit for Final Approval** | Yes (from In Implementation, own record only) | No | No | No |
| **Submit Decision (Final)** | No | Yes (from Pending Final Approval, assigned record only) | No | No |
| **View Any CC** | Yes | Yes | Yes | Yes |
| **Manage Users** | No | No | No | Yes |

### 2.3.4 Navigation Visibility by Role

| Navigation Item | CC Owner | Approver | Viewer | Admin |
| --- | --- | --- | --- | --- |
| **Dashboard** | isible (shows Pending Approvals card, My Drafts card, and Overview stats)  | Visible (shows Pending Approvals card, My Drafts card, and Overview stats)  | Visible (shows Pending Approvals card, My Drafts card, and Overview stats) | Visible (shows Pending Approvals card, My Drafts card, and Overview stats) |
| **All Change Controls** | Visible | Visible | Visible | Visible |
| **My Change Controls** | Visible (filtered to own records) | Visible (filtered to own records, if any) | Visible (typically empty) | Visible (typically empty) |
| **Approvals** | Visible (typically empty) | Visible (shows assigned pending items) | Visible (typically empty) | Visible (typically empty) |
| **Settings** | Visible (Profile tab only) | Visible (Profile tab only) | Visible (Profile tab only) | Visible (Profile + User Management tabs) |
| **"+ Create Change Control" Button** | Visible | Hidden | Hidden | Hidden |

**Note:** The Pending Approvals and My Drafts cards are displayed to all roles. They show relevant records when the user has items requiring action, or an empty state message ("No pending approvals" / "No drafts yet") when they do not.

---

### 2.4 Segregation of Duties

Segregation of duties is a mandatory compliance principle enforced by the Change Control module. It ensures that the person requesting and implementing a change is not the same person who approves it.

### 2.4.1 Core Rule

**The CC Owner and the Approver on any given Change Control record must be different individuals.** This rule is enforced by the system backend — it is not merely a procedural guideline.

### 2.4.2 Enforcement Mechanisms

**Single Role Per User:**
Every user in the system is assigned exactly one role at any time. A user cannot hold multiple roles simultaneously. Therefore, a user who has the CC Owner role cannot appear in the Approver dropdown (which only shows users with the Approver role), and vice versa.

**Approver Dropdown Filtering:**
The "Assign Approver" dropdown only displays users who currently hold the Approver role. Since a CC Owner can only hold the CC Owner role, they will never appear in this dropdown.

**Role Change Restriction for Active Records:**
To prevent a scenario where a user creates a CC as a CC Owner and then has their role changed to Approver (which could allow them to appear in Approver dropdowns for their own record), the system enforces the following rule:

An Admin cannot change a user's role if that user is associated with any active Change Control records. A record is considered "active" if it is in any state other than Closed or Cancelled. The association applies when the user is either the CC Owner of the record or the assigned Approver on the record.

When the Admin attempts to change a user's role and active records exist, the system shall block the role change and display an error message identifying the active CC-IDs that must be resolved (closed or cancelled) before the role change can proceed.

**Rationale:** This approach prevents segregation of duties violations at the source — the role-assignment level — rather than requiring complex per-record validation at submission time. The edge case of "CC Owner becomes Approver on their own record" is structurally impossible under this rule.

### 2.4.3 What Segregation of Duties Prevents

- A user creating a CC and then approving their own CC
- A user bypassing independent review by assigning themselves as the Approver
- A single individual having unchecked authority over the full change lifecycle

### 2.4.4 Record-Level Ownership Distinction

It is important to note that ownership and permissions are **record-specific**, not role-wide:

- If User A and User B both hold the CC Owner role, and User A creates CC-001, only User A is the owner of CC-001. User B cannot cancel CC-001 or perform owner-specific actions on it, even though User B also has the CC Owner role.
- The "Cancel CC" action is restricted to the CC Owner **of that specific record**, not to any user with the CC Owner role.
- The "Submit for Approval," "Submit for Final Approval" actions follow the same record-ownership principle.

This ensures that record ownership is tied to the individual who created the record, not to the role in general.

---