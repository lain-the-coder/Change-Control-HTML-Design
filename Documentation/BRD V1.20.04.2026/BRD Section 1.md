# Section 1 - Reviewed

---

## 1. EXECUTIVE SUMMARY

### 1.1 Project Overview

EAMI QMS (Quality Management System) is a web-based platform being developed to support structured quality and compliance management across the organisation. The platform is designed as a modular system, with each module addressing a specific quality management function.

This Business Requirements Document (BRD) defines the requirements for the **Change Control module** — the first module to be developed within the EAMI QMS platform. The Change Control module provides a formal, auditable process for requesting, evaluating, approving, implementing, and closing changes that affect systems, processes, infrastructure, or operations.

The module enforces a structured six-state workflow with two approval gates, role-based access control across four user roles, and field-level permissions governing who can edit which fields at each stage of the lifecycle. The system manages 50 fields across 11 form sections, with a comprehensive security matrix defining permissions by role and state.

This document serves as the **single source of truth** for all business requirements related to the Change Control module. It is intended to be comprehensive enough that any business question arising during design, development, testing, or deployment can be answered by referring to this document.

### 1.2 Business Objectives

The Change Control module addresses the following business objectives:

**BO-1: Formalise the Change Management Process**
Replace informal or ad-hoc change management practices with a standardised, structured workflow that ensures every change follows a consistent evaluation and approval process before implementation.

**BO-2: Enforce Segregation of Duties**
Ensure that the person requesting and implementing a change is never the same person approving it. The system enforces that the CC Owner and Approver must be different individuals, maintaining independence in the approval process.

**BO-3: Maintain a Complete Audit Trail**
Capture and permanently retain a record of all significant actions, state transitions, approvals, rejections, and critical field changes. This supports regulatory compliance, internal audits, and dispute resolution.

**BO-4: Provide Structured Risk Assessment**
Require risk evaluation at the approval gate, where the Approver independently assesses and assigns a risk level to each change. This ensures that risk is evaluated by a reviewer rather than the change requestor.

**BO-5: Ensure Implementation Accountability**
Require the CC Owner to document implementation details, validation evidence, and any deviations from the original plan before the change can be closed. A second approval gate validates that implementation was completed satisfactorily.

**BO-6: Support Regulatory and Compliance Requirements**
Provide the documentation structure, approval controls, and audit capabilities needed to demonstrate compliance with quality management standards and regulatory expectations applicable to the organisation.

**BO-7: Establish the Foundation for the QMS Platform**
Deliver the first functional module of the EAMI QMS, establishing the user management framework, authentication model, navigation structure, and design patterns that subsequent modules (CAPA, Deviation, Risk Register) will build upon.

### 1.3 Scope

### 1.3.1 In Scope

The following capabilities are within the scope of this Change Control module (Phase 1):

**Workflow Management**

- Six-state lifecycle: Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed, and Cancelled
- Two approval gates with approval/rejection decision logic
- Rejection workflow that returns the record to the previous state for revision and resubmission
- Cancellation workflow available only from the Initiated state, requiring a mandatory cancellation reason

**Form and Field Management**

- 50 fields organised across 11 form sections: Change Details — Identification (6 fields), Change Details — Change Definition (6 fields), Change Details — Planning (4 fields), Impact & Risk Assessment (8 fields), Implementation Plan & Validation (4 fields), Implementation Details (6 fields), Approvals — Initiation (2 fields), Approvals — Implementation Approval (5 fields), Approvals — Final Approval (4 fields), Approvals — Status (3 fields), and Additional Information (2 fields)
- Field-level permissions governed by the Security Matrix, defining editable, read-only, and not-applicable states for each field by role and workflow state
- 13 system-generated fields that are always read-only for all users
- Mandatory field validation enforced at submission points

**Role-Based Access Control**

- Four user roles: CC Owner, Approver, Viewer, and Admin
- Field-level and action-level permissions enforced per the Security Matrix
- Segregation of duties: CC Owner and Approver must be different individuals on any given record

**User Management**

- Standalone user database managed within the application (no external directory integration)
- Admin-managed user creation, role assignment, and user deactivation via the Settings interface

**Notifications**

- Email notifications triggered at each state transition
- Task due dates communicated via email notifications (Approval task: Submission Date + 5 business days; Implementation task: Target Closure Date − 3 business days; Final Approval task: Target Closure Date)

**Audit Trail**

- Comprehensive audit logging of state transitions, critical field changes, approval and rejection decisions, cancellation reasons, and user management actions
- Audit data stored in a database table, retained indefinitely, never deleted
- Old field values preserved in the audit log when overwritten during re-review cycles

**Navigation and Views**

- Dashboard with action-required items and system-wide statistics
- All Change Controls list view (accessible to all roles)
- My Change Controls list view (filtered to the logged-in user's records)
- Approvals queue (filtered to items pending the logged-in Approver's review)
- Settings pages for profile management and user administration

**Authentication**

- Login with email and password
- Password reset via email
- Session timeout after 30 minutes of inactivity

### 1.3.2 Out of Scope

The following items are explicitly **not included** in Phase 1 and are documented as future enhancements in Section 13:

- Emergency or fast-track change workflow (no "Emergency" category)
- CC Owner delegation or ownership transfer for in-progress records
- Cross-module traceability (linking to CAPA, Deviation, or Risk Register)
- Stale record detection, auto-escalation, or automated reminder emails for overdue tasks
- Audit trail viewer in the user interface (audit data is captured in the database only)
- External directory integration (Azure AD, LDAP, or similar)
- Direct clickable links to CC records within email notifications
- Reporting or analytics dashboards beyond the basic dashboard statistics
- Mobile-specific or native application interfaces
- Multi-language support
- Digital signatures or e-signature integration
- Bulk operations (creating, approving, or closing multiple CCs simultaneously)

### 1.4 Success Criteria

The Change Control module will be considered successfully delivered when all of the following criteria are met:

**SC-1: Complete Field Implementation**
All 50 fields are functional, correctly validated, and display the appropriate permission state (editable, read-only, not applicable, or system-managed) based on the current workflow state and the logged-in user's role, as defined in the Security Matrix.

**SC-2: Workflow State Machine**
All six workflow states operate correctly with proper state transitions. The "Submit for Approval," "Submit Decision," "Submit for Final Approval," and "Cancel CC" actions trigger the correct transitions. Rejection at either approval gate returns the record to the appropriate previous state.

**SC-3: Role-Based Access Enforcement**
The four roles (CC Owner, Approver, Viewer, Admin) have the correct permissions enforced at both the field level and the action level. Segregation of duties prevents a user from being both the CC Owner and Approver on the same record.

**SC-4: Notification Delivery**
Email notifications are sent at every state transition with the correct task due dates. Notifications include the CC-ID and a summary of the required action.

**SC-5: Audit Trail Completeness**
All state transitions, critical field changes (Decision, Risk Level, Decision Comments, Final Decision, Final Comments, Cancellation Reason, Target Closure Date, Proposed Implementation Date, Assign Approver), approval/rejection events, and user management actions are captured in the audit log with correct timestamps and user attribution. Old values are preserved when fields are overwritten during re-review cycles.

**SC-6: Validation Enforcement**
All mandatory field validations are enforced at submission. Date validations enforce minimum lead times (Proposed Implementation Date ≥ 2 business days; Target Closure Date ≥ 10 business days). Target Closure Date is locked after initial submission.

**SC-7: Cancellation Integrity**
Cancellation is available only from the Initiated state, only to the CC Owner of that specific record, and requires a mandatory cancellation reason entered via a popup modal. Cancelled records are permanent, read-only, and retained indefinitely.

**SC-8: UI Consistency with Prototypes**
The implemented user interface matches the approved HTML prototypes in terms of layout structure, field organisation, section grouping, navigation elements, and field display patterns across all workflow states and role-based views.

**SC-9: Data Retention**
Change Control records, audit log entries, and user records are retained indefinitely with no automatic deletion. Cancelled records remain in the system with all fields in a read-only state.

**SC-10: Platform Foundation**
The user management framework, authentication model, and navigation structure are delivered in a manner that supports the future addition of QMS modules (CAPA, Deviation, Risk Register) without requiring structural rework.

---