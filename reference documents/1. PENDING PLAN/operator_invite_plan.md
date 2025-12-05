Operator Onboarding & Role Hierarchy Design Log
Date: November 27, 2025 Status: Design Phase / Pre-Implementation

This document serves as a detailed record of the design process for the Operator Onboarding workflow. It documents the shift from public registration to a vetted B2B "Request & Approve" model, including security measures and future scaling strategies.

1. The Core Problem: Onboarding in a Closed System
   Q: We removed the public register page. How do new Operators (like my boss Benjamin) join?
   Developer Doubt: I am the developer and have an account. But once the app is live, how does a new person actually get in if there is no "Sign Up" button?

Architectural Definition: "Invitation-Only" / "Chain of Trust"

Context: Operators are "Trusted Personnel" (School Principals, Security Heads). We cannot allow random public signups (risk of "Fake Police" circles).

The Solution: Move from Self-Signup to Request & Approve.

Immediate Mechanism (For Benjamin): As the "Root Admin", you don't ask him to sign up. You grant him access via an internal Admin Interface or by manually creating his seed account.

2. The "Request Access" Workflow (The Public Face)
   Q: When the app goes worldwide, how do random new organizations apply?
   Developer Doubt: We can't manually invite everyone. How does a stranger in Texas request to become an operator?

Architectural Decision: The "Partner Request" Page

Location: A new public route: /request-access.

Access: Linked from the Login page ("Don't have an account? Request Access").

The Form: Collects high-trust data:

Full Name

Work Email (e.g., @school.edu)

Organization Name

Reason/Use Case

Storage: Requests are saved to a temporary staging table: operator_requests. They are not added to the users table yet.

3. The Approval Workflow (The Admin Face)
   Q: Where do these requests go? How do I approve them?
   Developer Doubt: Do I need a whole separate Admin Portal?

Architectural Decision: Integrated Dashboard "Admin" Tab

UI: A new tab inside the existing Dashboard called "Admin".

Visibility: This tab is hidden for normal operators. It is only visible to Super Admins.

Functionality:

Lists all pending requests.

"Approve" Button: Automatically creates the user account, sets a temporary password, and emails the user.

"Reject" Button: Marks request as rejected.

4. Security & Roles (RBAC)
   Q: If an operator is approved, can they see pending requests? Who approves who?
   Developer Doubt: "I hope not all operators will see pending requests... A California operator shouldn't approve a Texas operator."

Architectural Decision: Role-Based Access Control (RBAC)

The Problem: A flat "User" table gives everyone equal power.

The Fix: Add a role column to the users table.

The Roles:

'operator': Can manage their specific Safety Circles. Cannot see the "Admin" tab.

'admin': (You, Benjamin). Can manage the platform, see the "Admin" tab, and approve requests.

Backend Protection: Implement requireAdmin middleware on all /api/admin/\* endpoints to prevent unauthorized API calls.

5. Scaling for the Real World (Regional Hierarchy)
   Q: In a worldwide scenario, 10 Super Admins can't verify 10,000 requests. How do we scale?
   Developer Doubt: "There needs to be location-wise Level 2 Super Admins... how do we design this hierarchy?"

Architectural Decision: Federated Governance Model (Phase 2)

Concept: Move from a Flat Admin model to a 3-Tier Hierarchy.

The Tiers:

Global Super Admin (Level 3): System Owners (You). Maintains code/billing.

Regional Admin (Level 2): Trusted partners (e.g., "SafeCast Texas"). They vet local requests.

Local Operator (Level 1): The actual users.

Routing Logic:

Update operator_requests to include a region (e.g., "US-TX").

Update users to include managed_region column.

The Query: When an Admin logs in, they only fetch requests that match their managed_region.

Decision for MVP: We will implement the Flat Model (Admin/Operator) first for the demo, but document this Regional Model as the scaling strategy.

Summary of Technical Implementation Steps
Database:

Create operator_requests table.

Alter users table to add role column.

Backend:

Create POST /api/auth/request-access (Public).

Create GET /api/admin/requests (Protected + Admin Only).

Create POST /api/admin/approve/:id (Protected + Admin Only).

Create requireAdmin middleware.

Frontend:

Build public RequestAccess.tsx page.

Update Dashboard.tsx to render the "Admin" tab conditionally based on user role.

Build the "Pending Requests" table component.
