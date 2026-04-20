# Section 9 - Reviewed

---

## 9. UI/UX GUIDELINES

This section defines the user interface and user experience principles, patterns, and standards that the Change Control module must follow. The 25+ HTML prototypes in the project files serve as the authoritative visual reference for layout, structure, and component behaviour. This section documents the design principles and patterns demonstrated in those prototypes so that the implemented application matches them consistently.

---

### 9.1 Design Principles

The Change Control module follows a flat, enterprise design aesthetic inspired by professional project management tools. The design prioritises clarity, efficiency, and information density over decorative elements.

**DP-1 — Clean and Professional:**
The interface uses a clean, minimal design with flat styling. There are no rounded cards with heavy shadows, no gradient backgrounds, and no Material Design aesthetics. The visual language is understated and professional, appropriate for a regulated quality management environment.

**DP-2 — Information Density:**
Forms and list views display information efficiently without excessive whitespace. Fields are grouped logically using section cards, and multi-column grid layouts are used where appropriate to reduce vertical scrolling and keep related fields visible together.

**DP-3 — Clarity of State:**
The current workflow state of a record must always be immediately visible to the user. The state is displayed as a status badge in the page header alongside the CC-ID, and is also shown in the Change Details — Identification meta-grid as the Current State field. The user should never need to guess which state a record is in.

**DP-4 — Role-Aware Interface:**
The interface adapts based on the logged-in user's role and their relationship to the specific record. Buttons, editable fields, and navigation items are shown or hidden based on role permissions. Users only see actions they are allowed to perform — they are not shown disabled buttons for actions they cannot take.

**DP-5 — Progressive Disclosure:**
Fields that are not yet relevant to the current workflow stage are displayed with "Not applicable" placeholder messages rather than being hidden entirely. This allows all users to understand the full scope of the form while making it clear which sections are currently active. The user can see what comes next without being overwhelmed by editable fields that don't apply yet.

**DP-6 — Consistent Patterns:**
The same field display patterns, button styles, layout grids, and interaction behaviours are used consistently across all states and role views. A user who learns the interface in one state should find it familiar in every other state.

---

### 9.2 Field Display Patterns

The Change Control form uses five distinct field display patterns to communicate the permission state of each field to the user. These patterns are applied consistently across all workflow states and role-based views.

### 9.2.1 Editable Field (Active Input)

Used when the field is editable by the current user in the current state. The field is rendered as an active form control that accepts input.

**Visual Characteristics:**

- Standard form control (text input, textarea, dropdown, date picker, or file upload)
- White background, standard border
- Active cursor and keyboard focus
- Mandatory fields marked with an asterisk (*) after the label
- Placeholder text providing guidance on expected input

**When Used:** Only for fields that the current user is permitted to edit in the current workflow state, as defined by the Security Matrix (green cells).

**Example Fields:** Change Title (CC Owner, Initiated state), Decision (Approver, Pending Implementation Approval state), Implementation Summary (CC Owner, In Implementation state).

---

### 9.2.2 Read-Only Field (Disabled Input)

Used when the field contains a value but is not editable by the current user in the current state. The field is rendered as a disabled form control displaying the existing value.

**Visual Characteristics:**

- Disabled form control (input, textarea, or dropdown with the `disabled` attribute)
- Greyed-out or muted background indicating non-editable status
- Cursor changes to indicate the field cannot be interacted with
- The existing value is clearly visible and readable

**When Used:** For fields that have been filled in a previous state but are now locked. Also used when a different role has edit access but the current user does not. Corresponds to red cells in the Security Matrix for fields that have a value.

**Example Fields:** Change Title (any user in any state after Initiated), Decision (CC Owner view in Pending Implementation Approval state), Actual Implementation Date (any user in Closed state).

---

### 9.2.3 Not Applicable Field (Placeholder Message)

Used when the field is not yet relevant to the current workflow stage. The field label is displayed, but instead of an input control, a contextual placeholder message explains why the field is not available.

**Visual Characteristics:**

- Field label displayed normally
- Instead of an input control, a styled placeholder div with a muted text message
- Message provides context about when the field will become available (e.g., "Not applicable — Available after approval")
- Distinct visual styling from both editable and read-only fields

**When Used:** For fields that belong to a future workflow stage. For example, Implementation Details fields in the Initiated state, or Final Approval fields in the In Implementation state.

**Standard Messages:**

- "Not applicable — Available after approval" (Implementation Details fields before approval)
- "Not applicable — Pending submission" (Approval decision fields before CC is submitted)
- "Not applicable — Will be set by approver during review" (Risk Level in CC Owner's Initiated view)
- "Not applicable — Pending implementation" (Final Approval fields before implementation)

**Example Fields:** Implementation Summary (any user, Initiated state), Decision (any user, Initiated state), Final Decision (any user, In Implementation state).

---

### 9.2.4 System-Managed Field (Meta Value)

Used for system-generated fields that are always read-only and are populated/managed by the system rather than by any user. These fields are displayed in a compact meta-grid layout distinct from the standard form layout.

**Visual Characteristics:**

- Displayed in a meta-grid layout (3-column grid for Identification fields)
- Label displayed in a smaller, muted style
- Value displayed as plain text (not inside an input control)
- No border, no input styling — clearly distinct from user-editable fields
- "—" (dash) used when the field has no value yet (e.g., approval timestamps before approval occurs)

**When Used:** For the 13 system-generated fields: CC-ID, Current State, Change Owner, Last Updated By, Created On, Last Updated On, Implementation Approval By/On, Final Approval By/On, Implementation Approval Status, Final Approval Status, and Actual Closure Date.

**Example Fields:** CC-ID ("CC-001"), Current State ("Initiated"), Change Owner ("John Doe"), Implementation Approval By ("—" before approval, "Jane Smith" after approval).

---

### 9.2.5 Conditional Visibility (Shown Only in Specific States)

Used for fields that are completely hidden in most states and only appear under specific conditions. Unlike "Not Applicable" fields (which show a placeholder), conditionally visible fields are not rendered at all until their display condition is met.

**Visual Characteristics:**

- Field is entirely absent from the form in states where it does not apply
- When visible, displayed as a read-only disabled textarea (since the value is already captured and cannot be edited)

**When Used:** Currently applies to one field only — Cancellation Reason (field #50).

**Cancellation Reason Visibility Rules:**

- Hidden in: Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed
- Visible in: Cancelled state only
- When visible: Displayed in the Additional Information section below the Comments field as a read-only textarea
- Value source: Captured via the cancellation popup modal during the cancellation action, not through an inline form control

---

### 9.3 Status Indicators & Colour Coding

### 9.3.1 Status Badge

Each Change Control record displays a status badge in the page header next to the CC-ID. The badge indicates the current workflow state using distinct background colours for immediate visual recognition.

**Status Badge Colours by State:**

| State | Background Colour | Text Styling |
| --- | --- | --- |
| Initiated | Light blue | Dark text |
| Pending Implementation Approval | Light amber/yellow | Dark text |
| In Implementation | Light purple | Dark text |
| Pending Final Approval | Light orange | Dark text |
| Closed | Light green | Dark text |
| Cancelled | Light red | Dark text |

**Display Format:** The status badge is rendered as an inline label positioned to the right of the page title (e.g., "Change Control: CC-001 `[Initiated]`").

### 9.3.2 List View Status Badges

In the All Change Controls, My Change Controls, and Approvals list views, each record row includes a smaller status badge showing the current state. The same colour scheme applies as in the form header, providing consistent visual cues across the application.

### 9.3.3 Role Badges (Admin Settings)

In the Admin Settings — All Users table, each user's role is displayed with a role badge. These badges use a distinct colour scheme from the workflow status badges to avoid confusion:

- Admin: Distinct styling (e.g., darker badge)
- End User roles (CC Owner, Approver, Viewer): Lighter, consistent styling

### 9.3.4 Action Button Styling

Buttons throughout the application follow a consistent colour-coded styling:

| Button Type | Style | Usage |
| --- | --- | --- |
| Primary (blue) | Solid background, white text | Positive workflow actions: "Submit for Approval," "Submit Decision," "Submit for Final Approval," "Create Change Control" |
| Secondary (grey) | Outline or muted background | Navigation and neutral actions: "Back to List," "Save Draft" |
| Danger (red) | Red background, white text | Destructive actions: "Cancel CC," "Confirm Cancellation" |

---

### 9.4 Form Layout Standards

### 9.4.1 Overall Page Structure

The application uses a two-panel layout:

- **Left panel:** Fixed sidebar containing the application logo ("EAMI QMS"), navigation links, and no collapsible behaviour
- **Right panel:** Scrollable main content area containing the page header, information banners, form sections, and action buttons

### 9.4.2 Form Section Cards

Each logical group of fields is wrapped in a section card with:

- A section title (h2 element, e.g., "Change Details," "Impact & Risk Assessment")
- Optional section subtitles (h3 element, e.g., "Identification," "Change Definition," "Planning") for subsections within a card
- Optional section notes for contextual information (e.g., "These fields will become available once the change is approved for implementation")
- Consistent padding and spacing between fields within the card

### 9.4.3 Grid Layouts

Fields within sections use responsive grid layouts to optimise horizontal space:

- **Meta-grid (3 columns):** Used for the Identification section's system-managed fields (CC-ID, Current State, Change Owner, etc.)
- **Grid-3 (3 columns):** Used for compact dropdown groups (e.g., Change Type, Change Category, Department/Function displayed side by side)
- **Grid-2 (2 columns):** Used for date field pairs (Proposed Implementation Date + Target Closure Date), time field pairs (Implementation Window Start + End), and approval timestamp pairs (Approval By + Approval On)
- **Full width (1 column):** Used for textareas, text inputs, and file upload fields that benefit from the full available width

### 9.4.4 Information Banner

The Initiated state form displays an information banner at the top of the main content area, below the page header:

- Blue/informational styling
- Content: "Before you submit — Fill out the change control details below. Fields marked with * are mandatory. Once submitted, this change will be sent for implementation approval."
- This banner is specific to the Initiated state and is not displayed in other states.

### 9.4.5 Form Actions Bar

Action buttons are positioned at the bottom of the form in a form actions bar with a two-sided layout:

- **Left side:** Navigation and destructive actions ("Back to List" button, "Cancel CC" button)
- **Right side:** Positive workflow actions ("Save Draft" button, "Submit for Approval" button)

The specific buttons displayed depend on the current state and the user's role. Only actions the user is permitted to perform are shown. In terminal states (Closed, Cancelled) and for non-participating roles (Viewer, Admin), only the "Back to List" navigation button is displayed.

### 9.4.6 Cancellation Modal

The cancellation modal is a centred popup overlay with:

- Semi-transparent dark background overlay (dimming the form behind it)
- White modal card with consistent padding
- Modal title: "Cancel Change Control"
- Confirmation message including the CC-ID
- Cancellation Reason textarea (mandatory, 500 character limit)
- Two-button layout: "Go Back" (grey, left-aligned) and "Confirm Cancellation" (red, right-aligned)

---

### 9.5 Navigation Structure

### 9.5.1 Sidebar Navigation

The sidebar provides the primary navigation for the application. It contains the following items, visible to all roles unless noted:

| Navigation Item | Description | Visibility |
| --- | --- | --- |
| **Dashboard** | Landing page with action-required items and overview statistics | All roles |
| **All Change Controls** | List of all CC records in the system, filterable and sortable | All roles |
| **My Change Controls** | Filtered list showing only the logged-in user's own CC records | All roles |
| **Approvals** | Queue of CC records pending the logged-in Approver's review | All roles (populated only for Approvers) |
| **Settings** | Profile management and user administration | All roles (User Management tab visible to Admin only) |

### 9.5.2 Dashboard Layout

The Dashboard is the default landing page after login. It is divided into two sections:

**Action Required Section:**

- **Pending Approvals card:** Displayed to all roles. Shows count and list of CC records pending the logged-in user's approval decision. Displays empty state ("No pending approvals") when the user has no items pending or does not hold the Approver role.
- **My Drafts card:** Displayed to all roles. Shows count and list of CC records owned by the logged-in user that are in the Initiated state (not yet submitted). Displays empty state ("No drafts yet") when the user has no draft records.

**Overview Section:**

- Displays system-wide statistics showing the count of CC records in each active state (Initiated, Pending Implementation Approval, In Implementation, Pending Final Approval, Closed).
- Each stat is displayed as a clickable card that navigates to the All Change Controls list.
- Visible to all roles.

The "+ Create Change Control" button is displayed in the page header area of the Dashboard, visible only to users with the CC Owner role.

### 9.5.3 List Views

The All Change Controls, My Change Controls, and Approvals views follow a consistent list layout:

- Table-style layout with columns for CC-ID, Change Title, Change Owner (or relevant actor), Current State (with status badge), and last updated date
- Clickable rows that navigate to the full CC form view
- Pagination for long lists
- Consistent sorting (most recently updated first by default)

### 9.5.4 Settings Pages

The Settings area contains two tabs:

**Profile Tab (all roles):**

- Displays the logged-in user's profile information (name, email, role)
- Allows basic profile management (e.g., password change)

**User Management Tab (Admin only):**

- Create New User section with fields for full name, email, password, and role selection
- All Users table showing all user accounts with name, email, role badge, and action buttons (edit, deactivate)
- Pagination for large user lists