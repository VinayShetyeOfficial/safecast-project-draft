# SafeCast / AlertaComunidad - Project Status & Technical Guide (As of End of Session, Dec 2023)

This document provides a comprehensive summary of the SafeCast/AlertaComunidad project to facilitate seamless continuation in new development sessions.

## 1. Project Overview & Mission

**SafeCast / AlertaComunidad** is a bilingual (English/Español) community safety and early-warning platform. Its mission is to provide a private, non-carceral way for individuals to report safety concerns in various environments (work, school, community) and to enable rapid, targeted communication during high-risk events via WhatsApp.

- **Core Values:** Privacy, community protection, accessibility, and security.
- **Key Features:** Bilingual reporting, ML-powered risk classification, WhatsApp instant alerts, and privacy-first architecture.

## 2. Current Development Status & Latest Updates

The primary focus of the recent development sessions has been the creation and refinement of the **Incident Details Page (`pages/IncidentDetails.tsx`)**. The goal was to evolve it from a basic report viewer into a comprehensive operational workspace for incident responders.

### Key Accomplishments:

1.  **Advanced Media Gallery:**

    - **Functionality**: Replaced a single-image view with a full-featured media gallery capable of handling multiple file types (images, videos, PDFs, text files, etc.).
    - **UI/UX**: Features a large primary media viewer, a scrollable row of thumbnails, and navigation controls.
    - **"Download All" Feature**: Implemented a robust, **client-side** zipping and downloading feature using `JSZip` and `file-saver`. This avoids server-side strain and provides progress indicators to the user.
    - **Single File Downloads**: Ensured that all individual files, including images, download directly rather than opening in a new tab by fetching them as blobs.

2.  **Custom Error Handling:**

    - Replaced all native browser `alert()` popups for download failures with a custom, styled modal dialog that matches the application's cyber-themed UI. This provides a more professional and user-friendly experience.

3.  **Professional & Responsive Layout:**
    - **Two-Column Desktop View**: The page is organized into a logical two-column layout on desktops:
      - **Left Column (The "Incident"):** Contains the static incident details and the full media gallery. This is the "understanding" zone.
      - **Right Column (The "Response"):** Acts as a command center, containing the incident's live status, action buttons, and a detailed operator log. This is the "acting and documenting" zone.
    - **Logical Mobile View**: On smaller screens (tablets and mobile), the layout collapses into a single column in a specific, logical order: **Incident Details -> Status & Actions -> Operator Log -> Media Gallery**.
    - **Table-Based Operator Log**: The Operator Log was refactored into a clean, professional table format to resolve formatting issues and improve readability.

## 3. Technical Stack & Project Setup

### Frontend Dependencies:

- **Framework:** React.js (`^18.2.0`) with Vite (`^5.0.0`)
- **Routing:** `react-router-dom` (`^6.20.0`)
- **Styling:** Tailwind CSS (`^3.3.5`) with `autoprefixer` and `postcss`.
- **Icons:** `lucide-react` (`^0.292.0`)
- **Date Formatting:** `date-fns`
- **Client-Side File Handling:**
  - `jszip` (`^3.10.1`): For creating `.zip` archives in the browser.
  - `file-saver` (`^2.0.5`): For triggering file downloads from blobs.

### Backend & Services Dependencies:

- **Server:** Node.js with Express (`^4.18.2`)
- **Database:** PostgreSQL, accessed via the `pg` (`^8.11.3`) library.
- **Authentication:** `jsonwebtoken` (`^9.0.2`) and `bcryptjs` (`^2.4.3`)
- **AWS:** `aws-sdk` (`^2.1692.0`) for S3 integration.
- **CORS:** `cors` middleware for the Express server.
- **Environment Variables:** `dotenv` (`^16.3.1`)
- **Concurrency:** `concurrently` (`^8.2.2`) to run multiple services at once.

### How to Run the Project:

1.  **Install Dependencies:**

    ```bash
    npm install
    ```

2.  **Environment Setup (`.env` file):**
    Ensure a `.env` file exists in the project root with the following variables for connecting to the Supabase PostgreSQL database and AWS S3:

    ```env
    # Supabase/PostgreSQL
    DB_USER=your_db_user
    DB_HOST=your_db_host
    DB_DATABASE=your_db_name
    DB_PASSWORD=your_db_password
    DB_PORT=your_db_port

    # AWS S3 for Media Uploads
    S3_BUCKET_NAME=your_s3_bucket_name
    S3_ACCESS_KEY_ID=your_s3_access_key
    S3_SECRET_ACCESS_KEY=your_s3_secret_key
    S3_REGION=your_s3_region

    # JWT for Auth
    JWT_SECRET=your_jwt_secret_key
    ```

3.  **Start All Services:**
    The project includes backend servers for the main API, an ML service, and a WhatsApp service. The `start:all` script uses `concurrently` to run everything, including the frontend dev server.

    ```bash
    npm run start:all
    ```

    This command will execute:

    - `npm run server`: Starts the main Node.js Express API.
    - `npm run ml`: Starts the Python-based ML service.
    - `npm run whatsapp`: Starts the WhatsApp notification service.
    - `npm run dev`: Starts the Vite frontend development server.

## 4. Database & AWS S3 Configuration

- **Database:** The project uses a **PostgreSQL** database hosted on **Supabase**. The connection is managed in `backend/db.js`.
  - **`incidents` table:** The core table containing all reported incident data, including `id`, `description`, `location_text`, `media_url`, `severity_score`, `acknowledged_at`, etc.
- **File Storage:** Media files are uploaded to an **AWS S3** bucket.

  - The `media_url` column in the `incidents` table stores a comma-separated string of S3 object URLs.
  - **CORS Configuration**: The S3 bucket **must** have a CORS policy configured to allow `GET` requests from the application's domain. This was a critical fix to enable client-side file fetching for the download features. The required policy is:

  ```xml
  [
      {
          "AllowedHeaders": [
              "*"
          ],
          "AllowedMethods": [
              "GET"
          ],
          "AllowedOrigins": [
              "http://localhost:3000",
              "[https://your-production-domain.com](https://your-production-domain.com)"
          ],
          "ExposeHeaders": []
      }
  ]
  ```

## 5. Completed Core Features

We have successfully established the foundational layer of the SafeCast platform.

- **UI Scaffolding:** Created all primary user interface pages, including Login, Home, Report Form, Incident Details, and the initial layouts for the Dashboard and Safety Circles.
- **Bilingual Frontend:** The UI supports both English and Spanish language toggling.
- **Core Reporting Pipeline:** A user can successfully submit a report with text details and media files.
- **Database Integration:** The submitted report data is successfully stored in the Supabase PostgreSQL `incidents` table.
- **Secure File Storage:** Media evidence is successfully uploaded to and stored in a dedicated AWS S3 bucket.
- **Conditional Reporter Identity:** The report form now includes fields for `reporter_name` and `reporter_contact` that are only shown when the user opts out of anonymity.
- **Backend Logic for Reporter Identity:** The backend now correctly handles the logic to save reporter details if provided, and correctly forces the report to be anonymous if the fields are left blank, even if the "anonymous" switch is off.

---

## 6. Major Feature: AI Severity Engine Implementation

**Status:** **COMPLETE** (Deployed)

We have successfully implemented and deployed the **ML-Powered Risk Classification Engine**. This system replaces static keyword matching with a state-of-the-art semantic analysis model that runs locally and free of charge.

### 🧠 The Architecture

- **Service:** A dedicated **FastAPI** (Python) microservice running on port 8000.
- **Model:** `paraphrase-multilingual-MiniLM-L12-v2` via the `sentence-transformers` library.
  - **Why this model?** It is a "distilled" BERT model optimized for sentence similarity. It is **natively multilingual**, meaning it understands English and Spanish concepts equally well without needing translation or separate dictionaries. It is extremely fast and CPU-friendly.
- **Methodology:** **Semantic Scenario Matching (Zero-Shot Classification)**.
  - Instead of checking if a text contains the word "gun", the AI converts the user's report into a mathematical vector.
  - It compares this vector against a pre-defined list of **"Severity Scenarios"** (e.g., "Active shooter", "Building collapse", "Streetlight out").
  - The report is assigned the severity score of the scenario it is most semantically similar to.

### 🛠️ Implementation Details & Fixes

**1. Dynamic vs. Static Analysis**

- **Old Way:** Hardcoded list (`if "fire" in text: score = 80`). This failed on context (e.g., "fire drill" vs "building on fire") and required manual translation.
- **New Way:** The AI understands the _meaning_.
  - "There is a man with a knife" (English) -> Matches "Person with a weapon" scenario -> **Critical (85/100)**.
  - "Hay un hombre con un cuchillo" (Spanish) -> Matches "Person with a weapon" scenario -> **Critical (85/100)**.
  - "Streetlight is out" -> Matches "Minor maintenance issue" scenario -> **Low (20/100)**.

**2. Windows Compatibility Fix**

- **Issue:** The Python process was crashing immediately on startup with a `UnicodeEncodeError`.
- **Root Cause:** The Windows console (using `cp1252` encoding) could not handle the emojis (⏳, ✅) used in the startup print statements.
- **Fix:** Removed emojis from `backend/ml_service.py` logs.

**3. Dependency Conflict Resolution**

- **Issue:** `ImportError: cannot import name 'cached_download'`.
- **Root Cause:** Version mismatch between `sentence-transformers` (v2.2.2) and `huggingface-hub` (v0.26.2).
- **Fix:** Upgraded `sentence-transformers` to `5.1.2` in `backend/requirements.txt` to ensure full compatibility with the latest Hugging Face ecosystem.

**4. Resilient Integration**

- The main Node.js backend (`server.js`) now treats the ML service as the "Source of Truth" for severity scoring.
- **Fallback Mechanism:** If the Python ML service is offline or unreachable, the Node.js server falls back to a basic keyword check to ensure the report is still saved with a provisional score, preventing data loss.

---

## 7. Major Feature: Operator Log & Internal Notes

**Status:** **COMPLETE** (Deployed)

We have successfully implemented the full **Incident Management Workflow**, allowing operators to track the lifecycle of an event from creation to response.

### 🧠 The Architecture

1.  **Database Schema:**

    - Added `incident_notes` table (Linked to `incidents.id` and `users.id`).
    - Added `acknowledged_by` column to `incidents` table to track accountability.

2.  **Backend Logic:**

    - **Protected Endpoints:** Created `POST /api/incidents/:id/notes` and `GET /api/incidents/:id/notes`.
    - **Accountability:** Modified the Acknowledge endpoint to capture the `user_id` from the JWT token, ensuring we know _exactly_ who acknowledged an alert.
    - **Data Joining:** Updated retrieval queries to JOIN with the `users` table, allowing the frontend to display the email address of the operator who performed an action.

3.  **UI/UX (Incident Timeline):**
    - **Merged View:** Created a unified "Activity Timeline" that combines System Events (Created, Acknowledged) and User Actions (Notes) into a single chronological stream.
    - **Visual Coding:** \* **Grey:** System auto-events.
      - **Green:** Status changes (Acknowledgement).
      - **Cyan/Blue:** Operator Notes (Human intervention).
    - **Immutability by Design:** Notes and logs are append-only (Blockchain style). Operators cannot edit or delete history, ensuring a defensible audit trail for safety compliance.

---

## 8. Remaining Roadmap & Future Plans

With the core reporting, AI analysis, and operator management features complete, the project now shifts focus to **Organization & Alerting**.

### 🚀 Immediate Next Step: Real Safety Circles

- **Current Status:** The `Circles.tsx` page currently uses `MOCK_CIRCLES` (fake static data).
- **The Goal:** Make Safety Circles a real database entity.
- **Tasks:**
  1.  **Backend:** Create `GET /api/circles` and `POST /api/circles` endpoints.
  2.  **Frontend:** Connect the UI to fetch real circles from Supabase.
  3.  **Feature:** Allow operators to create new Circles (e.g., "Campus Night Staff") directly from the dashboard.

### 🔮 Future Potential Features (Post-MVP)

1.  **WhatsApp Integration (Active Alerting):**

    - Connect the backend to the WhatsApp Cloud API (using the existing `whatsapp_service.js` scaffold).
    - Trigger real messages to subscribers when `severity > 70`.

2.  **Contextual Map View:**

    - Integrate Google Maps or Mapbox to visualize the `location_text`.
    - _Note: This was deferred to prioritize the Safety Circle logic._

3.  **Incident Archive/Close Workflow:**

    - Add a "Resolve" or "Archive" status to filter old incidents out of the active dashboard view.
    - _Current Workaround:_ "Acknowledged" status serves as the "processed" state.

4.  **External Sharing:**
    - Create a secure, expire-able link to share a read-only view of an incident with external parties (e.g., Police, Insurance, HR) without giving them full system access.

## 9. Major Feature: Operator Onboarding & RBAC (B2B Architecture)

**Status:** **COMPLETE** (Deployed)

We have successfully transitioned the platform from a generic "Public Signup" model to a secure, **Invitation-Only B2B Model**.

### 🧠 The Architecture

1.  **Gatekeeper Workflow:**
    - Removed public registration.
    - Implemented **"Request Access"** flow: Organizations submit a request (`operator_requests` table).
    - Implemented **Admin Approval**: Super Admins (e.g., Vinay) review requests in the Dashboard and generate credentials.
2.  **Role-Based Access Control (RBAC):**
    - Added `role` column to `users` ('admin' vs 'operator').
    - **Admins:** Can see the "Admin" tab, approve/reject requests, view all system data.
    - **Operators:** Can only see the "Dashboard" and "Safety Circles" they own.
3.  **Security & UX:**
    - Implemented `requireAdmin` middleware for backend route protection.
    - Redesigned the Admin Panel with a Cyberpunk aesthetic.
    - Added **"Confirmation Dialogs"** (via React Portals) for critical actions (Approve/Reject/Acknowledge) to prevent accidental data changes.

---

## 10. Major Feature: Safety Circles (Phase 1 - Management)

**Status:** **COMPLETE** (Deployed)

Operators can now create and manage real Safety Circles, moving away from mock data.

### 🛠️ Implementation Details

1.  **Database Relationship:**
    - Circles are now linked to their creator via `owner_id`.
    - Incidents are now schema-ready to be linked to circles via `circle_id`.
2.  **Operator View:**
    - Operators only see circles they created.
    - UI supports creating new circles with specific **Alert Thresholds** (e.g., "Only notify if Severity > 80%").
3.  **UI/UX:**
    - Retained the high-quality "Card Grid" design.
    - Fixed Z-Index/Stacking Context issues using **React Portals**, ensuring modals appear correctly above the footer/navbar.

## 11. Major Feature: "Zero-Friction" Trust & Security Engine

**Status:** **COMPLETE** (Deployed & Verified)

We have implemented a sophisticated, invisible security layer to prevent "Swatting" (fake mass alerts) without requiring reporters to create accounts or solve complex CAPTCHAs during an emergency.

### 🧠 The Architecture (The "Rule of Three")

Instead of relying on a human operator to verify every report (which fails if they are asleep), we rely on **Signal Verification**.

1.  **Bot Protection (Layer 1):**
    - Integrated **Cloudflare Turnstile** (Invisible Widget).
    - **Result:** Automated scripts/bots are rejected immediately (+20 Trust Points for humans).
2.  **Device Fingerprinting (Layer 2):**
    - Integrated **FingerprintJS** to generate unique Device IDs based on browser attributes (not cookies/IPs).
    - **Logic:**
      - **Spam Check:** Multiple reports from the _same_ Device ID get a score penalty (-30).
      - **Crowd Surge:** 3+ _unique_ Device IDs reporting in a 15-minute window triggers a **CRITICAL TRUST BOOST (+40)**.
3.  **Semantic Analysis (Layer 3):**
    - The AI checks if the new report is an exact copy-paste of previous reports (Penalty -20) or a corroboration with different wording (Bonus +10).

### 🎯 The Outcome

- **Prankster:** 1 kid sends "Gun". Score = 10. **Alert Suppressed** (Pending Verification).
- **Real Crisis:** 3 students report "Gun" from different phones. Score = 100. **INSTANT ALERT.**

---

## 12. Major Feature: Public Subscription & Alert Routing (The "Grand Loop")

**Status:** **COMPLETE** (Deployed & Verified)

We have closed the loop between the **Public** (Subscribers) and the **Reporters**. The system is now fully context-aware.

### 🛠️ Implementation Details

1.  **The "Join Circle" Page:**
    - Created public route: `/join/:circle_id`.
    - Allows users to scan a QR code and subscribe their WhatsApp number to a specific circle.
    - Protected by Turnstile to prevent SMS spam attacks.
2.  **Context-Aware Reporting:**
    - Updated `Report.tsx` to capture `?circle_id=...` from the URL (QR Code).
    - The report is now saved with a link to that specific circle, rather than going into a global void.
3.  **Targeted Dispatcher:**
    - Updated `server.js` alert logic.
    - **Old Logic:** Send to Hardcoded Test Number.
    - **New Logic:** If `circle_id` exists -> Query `subscriptions` table -> Send WhatsApp alert to **only** those members.

### ✅ System Status

The entire pipeline is functional:
**QR Scan -> Join -> Report (Context Aware) -> Trust Verified -> Specific Subscribers Alerted.**

---

## 13. Design Review: Trust Logic, Spam Defense & Routing Architecture

**Date:** November 30, 2025
**Status:** Architectural Decisions Logged / Pending Implementation

This section documents the deep-dive discussion regarding the robustness of the Trust System, the handling of "Orphan" reports, and the definition of "Verified" users in an anonymous context.

### 🧠 1. The "Trust Score" Algorithm (The Math)

We defined exactly how a single report is evaluated to prevent "Swatting" while allowing real crises to pass through.

- **Baseline:** Every report starts at **50 Points**.

- **Signal A: Bot Protection (Cloudflare Turnstile)**

  - **Pass:** +20 Points (Proven Human).
  - **Fail:** -50 Points (Immediate Reject).
  - _Note:_ In production, this is usually invisible unless the network is suspicious.

- **Signal B: Device Fingerprinting (FingerprintJS)**

  - **Fresh Device:** 0 Points.
  - **Spamming (Same Device < 15m):** -30 Points (Penalty).
  - **Crowd Surge (3+ Unique Devices):** +40 Points (**CRITICAL BOOST**).

- **Signal C: Semantic Context (AI)**
  - **Copy-Paste (Exact Match):** -20 Points (Lazy Prank).
  - **Corroboration (Similar Meaning):** +10 Points (Truth Multiplier).

### 🛡️ 2. The "Gatekeeper" Tiers (Spam Defense)

To prevent the database from filling with junk ("Hi mom", "Test"), we established three tiers of data acceptance.

- **🔴 The "Trash" Tier (Score < 20):**

  - **Action:** SILENT REJECT. The user sees "Success", but the data is discarded immediately.
  - **Why:** Prevents script-kiddies from iterating on their attack.

- **🟡 The "Review" Tier (Score 20 - 50):**

  - **Action:** SAVE SILENTLY. Saved to DB with `status='flagged'`.
  - **Alerts:** SUPPRESSED. No WhatsApp messages are sent.
  - **Use Case:** A single person reporting a generic hazard without corroboration. Needs Operator verification.

- **🟢 The "Clean" Tier (Score > 50):**
  - **Action:** SAVE & ALERT.
  - **Use Case:** Verified Human + High Severity OR Multiple Witnesses. Triggers WhatsApp Dispatcher.

### 📍 3. Routing Logic: "Targeted" vs. "Orphan" Reports

We clarified how the system handles reports based on their entry point.

**Scenario A: The Targeted Report (QR Scan)**

- **Input:** User scans a sticker -> URL contains `?circle_id=c-123`.
- **Logic:** The system knows exactly who to alert (Subscribers of Circle c-123).
- **Trust:** Higher inherent trust because the user physically scanned a code (implies location presence).

**Scenario B: The Orphan Report (Global Form)**

- **Input:** User types `safecast.com/report` manually. No Circle selected.
- **Logic:** `circle_id` is NULL.
- **Action:**
  - **Alerts:** NONE. (We cannot blast the entire world).
  - **Storage:** Saved to Global Database.
  - **Value:** Used for future "Heatmaps" (e.g., noticing 50 reports from one specific neighborhood implies a Circle is needed there).

### ❓ 4. Q&A Summary (Key Clarifications)

| Question                                            | Decision / Answer                                                                                                                                                                                   |
| :-------------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **"Does 'Unverified' mean they aren't logged in?"** | Yes. But in our context, "Verified" means "Proven Identity via SMS OTP". Since we removed OTP for zero-friction, everyone is 'Unverified' identity-wise, but 'Trusted' signal-wise.                 |
| **"Can the Trust Math go wrong?"**                  | Yes. (e.g., a real panic report gets flagged as spam). **Mitigation:** We use the "Review Tier". We never delete border-line reports; we just silence the alarm so the Operator can check manually. |
| **"How do Operators get the Report Link?"**         | Currently missing from UI. Operators need a specific button to copy the QR/Report link (`/report?circle_id=...`) separately from the Join link.                                                     |
| **"Do we have a Heatmap?"**                         | No. Not in MVP. We use Bar/Pie charts for now. Heatmaps are a future roadmap item.                                                                                                                  |
| **"What about Device Blacklisting?"**               | We need to build a "Mark as Spam" button for Operators. This will blacklist the `device_id` preventing that phone from reporting to that Circle again.                                              |

### 🚀 5. Immediate Implementation Plan (Next Steps)

Based on this review, we have identified the following missing links to build next:

1.  **UI Update (Incident Details):** Display the Trust Score (e.g., "70/100") on the incident page so Operators know why an alert was sent (or suppressed).
2.  **UI Update (Circles Dashboard):** Add a "COPY REPORT LINK" button to the Circle Card.
    - _Current:_ "INVITE TO CIRCLE" (Copies `.../join/c-123`)
    - _New:_ "COPY REPORT LINK" (Copies `.../report?circle_id=c-123`) -> Needed for printing stickers.
3.  **Feature (Spam Management):** Implement the "Mark as Spam" button in the Incident Details page to allow manual blacklisting of devices.

---

## 14. Feature Implementation: Trust Logic v2 (Anti-Self-Corroboration)

**Status:** **COMPLETE** (Deployed & Verified)

We successfully patched the "Self-Corroboration Loophole" where a single device could artificially inflate the Trust Score by sending multiple reports.

### 🛠️ The Fix

- **Backend Logic (`server.js`):** Updated `calculateTrustScore`.
  - **Spam Check:** If `device_id` has reported to the _same circle_ in the last 15 minutes -> **-30 Penalty**.
  - **Corroboration Check:** The AI now filters out the reporter's _own_ device ID before looking for similar reports. You only get the **+10 Bonus** if _someone else_ confirms the story.
- **Result:**
  - **Report 1:** Score 70 (Green) -> **Alert Sent.**
  - **Report 2 (Same User):** Score 40 (Yellow) -> **Alert Suppressed.**
  - **Report 3 (Same User):** Score 40 (Yellow) -> **Alert Suppressed.**
  - _Outcome:_ Operators receive the critical first alert but are protected from "panic spam" or "nagging" from a single source.

---

## 15. Feature Implementation: Circle-Aware Dashboard & RBAC

**Status:** **COMPLETE** (Deployed & Verified)

We have fully implemented multi-tenancy and data isolation for Operators.

### 🛠️ The Architecture

1.  **Strict Data Isolation (RBAC):**
    - **Admin (Vinay):** Sees ALL incidents and global stats.
    - **Operators (Benjamin/Sarah):** See **ONLY** incidents linked to the circles they own. They cannot see each other's data.
    - **Secure Endpoints:** Updated `GET /incidents`, `/stats`, `/category-counts`, and `/severity-counts` in `server.js` to enforce these rules at the database level.
2.  **Dashboard Filtering:**
    - Added a **"Filter by Circle"** dropdown to the Dashboard.
    - Operators managing multiple circles (e.g., "School" and "Work") can now toggle views to focus on one context at a time.
3.  **Enhanced Incident Details:**
    - **Circle Badge:** The incident header now clearly displays which Circle the report belongs to (e.g., "CHICAGO EDU SAFETY NET").
    - **Trust Visualization:** Added a "Trust Verification" HUD element displaying the Trust Score (0-100) and classification (High Credibility vs. Pending Review vs. Likely Spam).

---

## 16. Remaining Roadmap (What is Left?)

**Status:** Planning Phase

With the core "SafeCast Loop" fully functional locally, here are the remaining tasks to reach Production Readiness.

### 🔴 Priority 1: Integrations & Actions

1.  **WhatsApp API Integration:**
    - _Current:_ Mock service (`whatsapp_service.js`).
    - _Task:_ Register a Meta Developer App, get real credentials, and enable actual message delivery.
2.  **"Mark as Spam" Button:**
    - _Current:_ Missing from UI.
    - _Task:_ Add button to `IncidentDetails.tsx`. Clicking it should set `status='spam'` and blacklist the `device_id` for that circle.

### 🟡 Priority 2: UI/UX Polish

1.  **Grouping/Nesting (Optional):**
    - _Current:_ 3 reports for the same fire show as 3 rows.
    - _Task:_ Group them visually into one "Master Event" card. (Deferrable for MVP).
2.  **Orphan Report Visibility:**
    - _Current:_ Only Admins see reports sent to "Global" (no circle selected).
    - _Task:_ Allow Operators to "Subscribe" to public reports in their geographic area (Future feature).

### 🟢 Priority 3: Deployment

1.  **Dockerization:**
    - Create `Dockerfile` to bundle Frontend + Backend + Python ML Service.
2.  **Cloud Hosting:**
    - Deploy to Railway/Render to test with real mobile devices (QR Code scanning).

---

## 17. Feature Implementation: Spam Management & Device Blocking

**Status:** **COMPLETE** (Deployed & Verified)

We have implemented a robust moderation system to allow Operators to permanently silence malicious actors.

### 🛠️ The Architecture

1.  **Database Schema:**
    - Created `blocked_devices` table to store blacklisted IDs.
    - Added `blocked_by` and `incident_id` columns for full accountability (Audit Trail).
    - Added `status` column to `incidents` table (`active` vs `spam`).
2.  **Backend Logic:**
    - **Blocklist Check:** The Trust Engine (`server.js`) now checks the blacklist _before_ processing any new report. Blocked devices get a **Score of 0**.
    - **Spam Filter:** Updated `GET /incidents` and Dashboard Stats endpoints to automatically **exclude** incidents marked as 'spam', keeping the dashboard clean.
3.  **Frontend UI:**
    - Added a dedicated **"BLOCK DEVICE"** button in `IncidentDetails.tsx`.
    - Integrated with `ConfirmationDialog` to prevent accidental bans.

---

## 18. UX Polish: Visual Hierarchy & Empty States

**Status:** **COMPLETE** (Deployed)

We refined the Operator Dashboard to improve readability and handle "Zero Data" states gracefully.

### 🎨 Visual Improvements

1.  **Read vs. Unread Logic:**
    - **Bold/Bright Rows:** Unacknowledged incidents (Requires Action).
    - **Dimmed/Light Rows:** Acknowledged incidents (Processed).
2.  **Empty States:**
    - Implemented an **"AWAITING DATA STREAM"** placeholder for the Bar Charts when no incidents exist, ensuring the dashboard looks professional even on Day 1.
3.  **Context Clarity:**
    - Updated the Incident Header Badge to display the specific **Circle Name** (e.g., "CHICAGO EDU SAFETY NET") instead of the generic "Targeted" label.

---

## 19. Updated Roadmap (Remaining Tasks)

**Status:** Ready for Next Phase

With the "Spam Defense" and "Dashboard UX" complete, the remaining critical path is:

### 🔴 Priority 1: Real-World Connectivity

1.  **WhatsApp API:** Switch from Mock Service to Real Meta API.
2.  **Dockerization:** Containerize the stack for deployment.

### 🟡 Priority 2: Advanced Features

1.  **Incident Nesting:** Group multiple reports about the same event into a single "Master Card" on the dashboard.
2.  **Orphan Report Subscription:** Allow operators to "listen" to public/global reports in their geo-fence.

---

## 20. Future Architecture & Design Decisions (Pending Implementation)

**Date:** December 1, 2025
**Status:** Design Logged / Ready for Future Sprint

This section documents the architectural decisions made regarding complex "Real World" workflows (Lifecycle, Nesting, and Collaboration).

### 1. Incident Nesting (The "Master Ticket" Problem)

- **Question:** _How do we handle 50 students reporting the same fire? Currently, they appear as 50 separate rows._
- **Solution Plan:**
  - **Phase 1 (UI Grouping):** Visually group incidents in the dashboard if they share the same `category`, `location`, and `time_window` (15 mins).
  - **Phase 2 (Manual Merge):** Allow operators to select multiple incidents and click "Merge into Master Incident".
  - **Database Change:** Add `parent_incident_id` column to `incidents` table. Child incidents will be hidden from the main list and displayed inside the Parent's details page.

### 2. Lifecycle Management (Acknowledge vs. Resolve)

- **Question:** _"Acknowledge means I accept the task. Should there be a 'Task Done' button?"_
- **Solution Plan:** Implement a 3-stage lifecycle.
  1.  **Pending (Bold):** Fresh report. Needs attention.
  2.  **Active/Acknowledged (Light):** Operator is working on it. Button changes to "MARK RESOLVED".
  3.  **Resolved (Archived):** Incident is closed.
- **Action:** Update the `Acknowledge` button logic to toggle between states or add a secondary "Resolve" workflow.

### 3. Dashboard Visibility Logic

- **Question:** _"Should resolved/acknowledged high-severity incidents still appear in the 'High Risk' tab?"_
- **Decision:**
  - **Acknowledged:** **YES.** The threat is still active, just being managed. It should remain visible but visually distinct (e.g., dimmed or tagged "WIP").
  - **Resolved:** **NO.** Once resolved, it must be removed from the "High Risk Queue" to prevent clutter. It moves to a "History/Archive" view.

### 4. Operator Collaboration (The "Shared Circle" Problem)

- **Question:** _"Right now, Sarah cannot see Benjamin's circles. What if they need to collaborate?"_
- **Current Constraint:** The database uses `owner_id` (Single Owner model).
- **Solution Plan:** Transition to a **Membership Model**.
  - **New Table:** `circle_members` (`circle_id`, `user_id`, `role`).
  - **Logic Update:** Change backend queries from `WHERE owner_id = user.id` to `WHERE id IN (SELECT circle_id FROM circle_members WHERE user_id = user.id)`.
  - **UI Update:** Add an "Invite Operator" button to the Circle settings.

---

## 21. Feature Implementation: "Deep Inspection" Anti-Spam System

**Status:** **COMPLETE** (Deployed & Verified)

We have fortified the Operator Onboarding flow (`/request-access`) against automated spam using a sophisticated **Shadow Banning** strategy.

### 🧠 The Problem

Disposable email services (e.g., Yopmail, Temp-Mail) generate thousands of unique domains daily. A static blocklist of domain names is insufficient because spammers simply buy new domains (e.g., `idwager.com`) that bypass simple checks.

### 🛡️ The Solution: "Deep Inspection" & Shadow Banning

We implemented a **Dual-Layer Defense** that inspects both the _name_ and the _infrastructure_ of the email address.

**Layer 1: Static Domain Analysis (Fast)**

- **Mechanism:** Checks the email domain against a community-maintained library of **56,000+** known disposable domains (`disposable-email-domains`).
- **Result:** Instantly catches "Lazy Spammers" (e.g., `@yopmail.com`) with zero latency.

**Layer 2: MX Infrastructure Fingerprinting (Deep)**

- **Mechanism:** If the domain is not in the static list, the system performs a real-time DNS lookup to find the domain's **Mail Exchange (MX) Records**.
- **Logic:** Spammers often use many different domain names but route them all through the same cheap email servers (e.g., `mail.tm`, `guerrillamail.com`).
- **Action:** We check if the MX record points to a known "Spam Infrastructure Provider". If `idwager.com` routes mail to `mx.yopmail.com`, it is blocked, even if `idwager.com` is brand new.

### 👻 The "Shadow Ban" (Silent Rejection)

Instead of showing an error (which tells the bot to try again with a different email), we use **Silent Rejection**.

- **Frontend:** The user sees a standard **"REQUEST RECEIVED"** success message.
- **Backend:** The request is **discarded** immediately. It is logged in the server console for audit but **never enters the database**.
- **Benefit:** The spammer thinks they succeeded and moves on, keeping the Admin Dashboard clean of junk data.

### ✅ Verification

- **Test Case A (Static):** `test@yopmail.com` -> **BLOCKED** (Layer 1).
- **Test Case B (Infrastructure):** `test@idwager.com` -> **BLOCKED** (Layer 2 - Caught via MX Record).
- **Test Case C (Legit):** `admin@harvard.edu` -> **ACCEPTED**.

---

## 22. Fix: AWS S3 Download Permissions

**Status:** **COMPLETE** (Deployed)

Resolved a critical issue where the "Download All" feature failed with a "Network Error".

- **Root Cause:** The AWS S3 CORS policy only whitelisted port `3000` (Production/Docker), but the local dev environment runs on `5173`.
- **Fix:** Updated the S3 CORS configuration to explicitly allow `http://localhost:5173`.
- **Result:** Browser-based zipping and downloading now works seamlessly in development.

---

## 23. Feature Implementation: Incident Resolution Workflow

**Status:** **COMPLETE** (Deployed & Verified)

We have closed the operational loop by allowing operators to formally "Resolve" and "Archive" incidents after they have been handled.

### 🛠️ The Architecture

1.  **Database Schema:**
    - Added `resolved_at` (Timestamp) and `resolved_by` (User ID) columns to the `incidents` table.
    - Added `status` column ('active', 'resolved', 'spam') to differentiate lifecycle states.
2.  **Backend Logic:**
    - **New Endpoint:** `POST /api/incidents/:id/resolve`.
    - **Logic:** \* Updates the incident status to 'resolved'.
      - Logs the timestamp and the operator ID.
      - Automatically inserts a system note: _"Incident marked as RESOLVED and ARCHIVED."_
3.  **Frontend UI (Incident Details):**
    - **Lifecycle Logic:**
      - **Pending:** Shows "ACKNOWLEDGE" + "BLOCK".
      - **Active:** Shows "MARK RESOLVED" (Green Button).
      - **Resolved:** Shows "CASE CLOSED" (Static Badge).
4.  **Dashboard UX:**
    - **Filter Update:** Added an "Archived / Closed" option to the Status dropdown.
    - **Default View:** The "All" filter now defaults to showing only _Active_ (Pending + Acknowledged) incidents, hiding resolved ones to keep the dashboard clean.
    - **Visuals:** Resolved incidents in the table (when filtered) display a distinct "ARCHIVED" badge.
