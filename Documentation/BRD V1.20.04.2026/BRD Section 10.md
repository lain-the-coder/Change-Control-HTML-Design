# Section 10 - Reviewed

---

## 10. NON-FUNCTIONAL REQUIREMENTS

This section defines the non-functional requirements that the Change Control module must satisfy. These requirements address how the system performs and operates rather than what it does functionally. They cover performance expectations, security and authentication, browser compatibility, and accessibility considerations.

---

### 10.1 Performance

**NFR-10.1.1 — Page Load Time:**
Standard pages (Dashboard, list views, CC form) shall load within a reasonable timeframe under normal operating conditions. The system should feel responsive and not introduce noticeable delays during typical use.

**NFR-10.1.2 — Form Submission Response:**
When a user clicks a submission button (Submit for Approval, Submit Decision, Submit for Final Approval, Save Draft, Confirm Cancellation), the system shall process the action and provide visual feedback (success confirmation or validation error) without excessive delay. The user should not be left uncertain about whether their action was processed.

**NFR-10.1.3 — Concurrent Users:**
The system shall support multiple users accessing the application simultaneously. Since the shared document model ensures that only one user has edit access at any given state, write conflicts are not expected. However, multiple users may be viewing different records or different states of the same record at the same time.

**NFR-10.1.4 — File Upload Performance:**
File uploads (Supporting Documents and Implementation Evidence) up to the 10MB maximum shall complete without timeout. The system shall provide visual feedback during upload progress.

**NFR-10.1.5 — List View Performance:**
The All Change Controls list view shall remain performant as the number of records grows. Pagination shall be used to manage large datasets and prevent excessive page load times.

**NFR-10.1.6 — Search and Filtering:**
List views should support filtering and sorting capabilities that execute within a reasonable timeframe, even as the volume of CC records increases over time.

---

### 10.2 Security & Authentication

### 10.2.1 Authentication

**NFR-10.2.1 — User Authentication Required:**
All pages and functions within the Change Control module shall require user authentication. Unauthenticated users shall not be able to access any application content. Unauthenticated requests shall be redirected to the login page.

**NFR-10.2.2 — Login Mechanism:**
The system shall provide a login page where users authenticate with their email address and password. The login page is the only publicly accessible page in the application.

**NFR-10.2.3 — Password Reset:**
The system shall provide a "Forgot Password" function that sends a password reset link to the user's registered email address. The reset link shall expire after a reasonable timeframe. The password reset page shall allow the user to set a new password.

**NFR-10.2.4 — Secure Password Storage:**
User passwords shall be stored securely using industry-standard hashing algorithms. Passwords shall never be stored in plain text.

**NFR-10.2.5 — Session Timeout:**
User sessions shall expire after 30 minutes of inactivity. When a session expires, the user shall be redirected to the login page and must re-authenticate to continue. Any unsaved form data may be lost on session timeout.

**NFR-10.2.6 — HTTPS Only:**
All communication between the user's browser and the application server shall be encrypted using HTTPS. Unencrypted HTTP connections shall not be permitted.

### 10.2.2 Authorization

**NFR-10.2.7 — Role-Based Access Control (RBAC):**
The system shall enforce role-based access control as defined in the Security Matrix (Section 4) and Action Permissions (Section 8.4). Every request that modifies data shall be validated against the user's role and their relationship to the specific record before the action is processed.

**NFR-10.2.8 — Server-Side Enforcement:**
All permission checks shall be enforced on the server side (backend). Client-side UI restrictions (hiding buttons, disabling fields) are a convenience for the user experience but shall not be the sole enforcement mechanism. A user who bypasses the client-side UI (e.g., using browser developer tools or API calls) shall still be blocked by server-side validation.

**NFR-10.2.9 — Record-Level Ownership Validation:**
For actions that are restricted to the record owner (Submit for Approval, Submit for Final Approval, Cancel CC), the backend shall validate that the authenticated user is the CC Owner of that specific record, not merely a user with the CC Owner role.

**NFR-10.2.10 — Assigned Approver Validation:**
For approval actions (Submit Decision at both gates), the backend shall validate that the authenticated user is the Approver assigned to that specific record, not merely a user with the Approver role.

**NFR-10.2.11 — Segregation of Duties Enforcement:**
Segregation of duties is enforced through the single-role-per-user model and the role change restriction for active records (BR-8.4.11). Since each user holds exactly one role at a time, a CC Owner cannot appear in the Approver dropdown. The system further prevents the edge case of a role change creating a conflict by blocking Admin from changing a user's role while they have active CC records. This structural approach eliminates the need for per-record ownership validation at approval submission time. See Section 2.4.2 and BR-8.4.11 for full details.

### 10.2.3 Data Protection

**NFR-10.2.12 — No Unauthorised Data Exposure:**
The system shall not expose CC record data, user data, or audit data to unauthenticated users or to users who do not have the appropriate role-based access. While all authenticated users can view all CC records (as per the business rules), the underlying APIs shall still validate authentication before returning data.

**NFR-10.2.13 — Audit Log Integrity:**
The audit log shall be protected from tampering. Application users (including Admins) shall not have the ability to modify, delete, or overwrite audit log entries through the application interface or API. The audit log is append-only.

---

### 10.3 Browser Compatibility

**NFR-10.3.1 — Modern Browser Support:**
The system shall function correctly on the current stable versions of the following browsers:

- Google Chrome
- Microsoft Edge
- Mozilla Firefox
- Apple Safari

**NFR-10.3.2 — No Legacy Browser Requirement:**
There is no requirement to support legacy browsers such as Internet Explorer. The application targets modern, standards-compliant browsers only.

**NFR-10.3.3 — Responsive Layout:**
The application shall be usable on standard desktop and laptop screen resolutions. While mobile-optimised or native mobile applications are out of scope (see Section 1.3.2), the layout should not break on common screen sizes.

**NFR-10.3.4 — JavaScript Required:**
The application may require JavaScript to be enabled for full functionality. This is standard for modern web applications and does not require a no-JavaScript fallback.

---

### 10.4 Accessibility

**NFR-10.4.1 — Basic Accessibility:**
The application shall follow basic web accessibility practices to ensure usability for a broad range of users. This includes semantic HTML structure, appropriate use of form labels associated with their input controls, sufficient colour contrast between text and background elements, and keyboard navigability for core workflows (form filling, button clicking, dropdown selection).

**NFR-10.4.2 — Form Labels:**
All form fields shall have associated labels that clearly identify the field's purpose. Mandatory fields shall be visually marked with an asterisk (*) and the mandatory status should be conveyed programmatically where feasible.

**NFR-10.4.3 — Error Messages:**
Validation error messages shall be displayed in a location clearly associated with the relevant field or action. Error messages shall use text descriptions (not colour alone) to communicate the nature of the error.

**NFR-10.4.4 — Status Communication:**
Status badges and colour-coded indicators shall include text labels in addition to colour. The workflow state is always communicated via the text label on the badge (e.g., "Initiated," "Closed"), not through colour alone.

**NFR-10.4.5 — No Full WCAG Compliance Requirement:**
Full compliance with WCAG 2.1 AA or any specific accessibility standard is not a Phase 1 requirement. However, the application should avoid introducing unnecessary accessibility barriers and should follow the basic practices described above as a foundation for future accessibility improvements.