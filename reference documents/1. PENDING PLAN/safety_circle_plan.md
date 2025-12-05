# Safety Circle Architecture & Design Log

**Date:** November 27, 2025  
**Status:** Design Phase / Pre-Implementation

This document serves as a detailed record of the design process for the Safety Circle feature. It chronicles the specific questions raised by the developer regarding real-world usage, edge cases, and scalability, along with the architectural solutions defined to address them.

## 1. Core Concept & Mechanics

**Q:** What exactly is a "Safety Circle"? How does it help? How do we keep track of members?
**Developer Doubt:** Is this just a WhatsApp group? How does the system manage it?

**Architectural Definition:**

- **Concept:** A Safety Circle is a private broadcast channel stored in the database, representing a specific community (e.g., "Downtown Watch", "Tech Park Security"). It acts as a filter to prevent "alert fatigue."
- **Benefit:** Users only receive alerts relevant to their specific context (e.g., a parent only gets school alerts, not factory alerts).
- **Membership Tracking:** It is not a WhatsApp group. It is a relational database link in the subscriptions table connecting a `user_id` (or anonymous sub) to a `circle_id` and their `whatsapp_number`.
- **Operator Role:** Operators configure the circle (set the Severity Threshold, e.g., "Only notify if Severity > 80") and manage invite links.

## 2. The Subscription Flow

**Q:** Reporters only see the Home/Report page. Operators see the Dashboard. How do users actually subscribe?
**Developer Doubt:** You mentioned QR codes, but what do they link to? A WhatsApp Group invite?

**Architectural Decision:**

- **No WhatsApp Groups:** Using actual WhatsApp groups would expose user phone numbers to strangers (privacy violation) and create noise.
- **The "Join" Page:** The QR code links to a public web page: `safecast.com/join/c-[circle_id]`.
- **The Flow:**
  1.  User scans QR code.
  2.  Browser opens the specific "Join Circle" page.
  3.  User enters their phone number.
  4.  System saves it to the subscriptions table.
- **Result:** The system sends 1-to-1 messages to these numbers when an alert triggers.

## 3. Geographic Scope & Routing

**Q:** Can this be used worldwide? If I live in Massachusetts, will I get alerts for Texas?
**Developer Doubt:** How does the system know "where" I am? Is a circle tied to a physical location?

**Architectural Decision:**

- **Scope:** The app is worldwide. Circles are defined by Organizational Geography, not just GPS.
- **Opt-In Logic:** You only get alerts for circles you explicitly join. A Massachusetts resident will never receive Texas alerts unless they scan a Texas QR code.
- **Operator Responsibility:** Operators create circles relevant to their scope (e.g., "Mass Bank HQ" vs. "Texas Bank Branch").

**Q:** Since the system doesn't store reporter details, how does a report get linked to the correct circle?
**Developer Doubt:** If I report a fire at "Mass Bank", how does the system know to alert the "Mass Bank Circle" and not the "Texas Circle"?

**Architectural Decision: "Context-Aware Reporting"**

- **Smart Links:** The QR code printed at the location contains the ID: `safecast.com/report?circle_id=123`.
- **Auto-Tagging:** When the user submits the report via that link, the `circle_id` is automatically attached to the incident in the database.
- **Routing:** The backend sees `circle_id=123`, looks up subscribers for #123, and alerts only them.

## 4. Resilience & Fallbacks

**Q:** What if the QR code sticker is torn down or missing?
**Developer Doubt:** A user in Texas wants to report an incident at a specific bank branch, but the sticker is gone. They only know the main website URL.

**Architectural Decision: "Circle Search" Feature**

- The Report Page will have an optional field: **"Reporting to a specific community?"**
- The user types "Texas Bank".
- A dropdown appears (fetched from backend): "Texas Bank - Austin", "Texas Bank - Dallas".
- Selecting one manually links the report to that circle ID.

**Q:** What if there is NO circle for the location (e.g., "Austin Beach")?
**Developer Doubt:** The user wants to report a hazard, but no specific circle exists. What happens?

**Architectural Decision: "Public/Orphan Report" Logic**

- **Action:** The user submits the report without selecting a circle.
- **Data State:** `circle_id` is NULL.
- **System Behavior:**
  - **Alerts:** No WhatsApp alerts are sent (0 subscribers).
  - **Storage:** The report is saved to the global database.
  - **Value:** These reports appear on a "Global Heatmap," signaling to authorities that a new Safety Circle might be needed in that area.

## 5. Operator Visibility (Multi-Tenancy)

**Q:** If an operator logs in, do they see EVERYTHING? Isn't that clutter and a security risk?
**Developer Doubt:** A California operator shouldn't see Florida incidents. We need to narrow this down.

**Architectural Decision: Circle-Based Access Control**

- **No "God View":** By default, operators cannot see the global feed.
- **The Rule:** An operator can only see incidents that belong to a Safety Circle they manage.
- **Database Changes:**
  - Link Incidents: `incidents` table gets a `circle_id` column.
  - Link Operators: New `circle_operators` table maps `user_id <-> circle_id`.
- **Dashboard Query:** Instead of `SELECT * FROM incidents`, the backend will query: `SELECT * FROM incidents WHERE circle_id IN (list_of_my_circles)`.
- **Public Report Handling:** Operators can optionally "subscribe" to Public Reports (`circle_id = NULL`) within a specific geographic text match (e.g., "Show me public reports containing 'California'").

---

# Safety Circle & Alert Trust Architecture Design Log

**Date:** November 29, 2025
**Status:** Architecture Finalized / Ready for Implementation

This document records the iterative design process for the Safety Circle Alerting System. It focuses specifically on the critical challenge of balancing **User Anonymity** with **System Security** (preventing hoaxes/swatting) in a public reporting environment.

## Phase 1: The Fundamental Mechanics

**Q: How do users actually join a circle? Is it just a WhatsApp group?**
**Developer Doubt:** "Is this just a WhatsApp group? How does the system manage it?"
**Design Decision: No WhatsApp Groups.**

- **Privacy Issue:** WhatsApp groups expose member phone numbers to strangers.
- **Solution:** Circles are database entities (`circles` table). Users subscribe via a public web page (`/join/:id`). The system sends 1-to-1 messages, keeping members anonymous to each other.

**Q: How do reporters link incidents to circles without an account?**
**Developer Doubt:** "If I report a fire at 'Mass Bank', how does the system know to alert the 'Mass Bank Circle' if I don't log in?"
**Design Decision: Context-Aware Links.**

- **Mechanism:** QR codes printed at the location contain the Circle ID (`safecast.com/report?circle_id=123`).
- **Fallback:** A "Manual Search" dropdown on the report page allows users to select a circle by name/location if they can't find a QR code.

## Phase 2: The "Swatting" Vulnerability (The Critical Pivot)

**Q: What stops a prankster from faking a crisis?**
**Developer Doubt:** "I [Third Person] go to safecast.com/report, search for a circle in the USA, and submit a HOAX High Alert. Will this not be a problem?"

- **The Threat:** A "Swatting" attack where a malicious actor triggers mass panic in a remote location using the public form.
- **Initial Idea: "Dark by Default"**
  - _Proposal:_ Remove public search. Make circles "Unlisted" so only people with the physical QR code or a specific "Access Code" can report.
- **Verdict:** **Insufficient.** Access codes can be leaked online. We need a stronger filter.

## Phase 3: The "Sleeping Operator" Problem

**Q: Can we rely on human verification?**
**Proposed Logic:** "If an anonymous report comes in, notify the Operator. They verify it, then blast the alert."
**Developer Doubt:** "Say the Operator is sleeping / using restroom. How can we overcome this and trigger high alerts instantly?"

- **The Constraint:** A "Human-in-the-Loop" creates a Single Point of Failure. In a live shooter scenario, seconds matter. If the operator is unavailable, the system fails.
- **The Pivot: "The Crowd-Surge Protocol"**
  - **New Logic:** We replace the Human Gatekeeper with a **Data Gatekeeper**.
  - **Concept:** One report might be a lie. Three reports are a crisis.
  - **Mechanism:** If the system detects **Volume (Multiple Reports)** in a short time window, it overrides the Operator and alerts automatically.

## Phase 4: The "Coordinated Attack" Vulnerability

**Q: What if the "Crowd" is fake?**
**Developer Doubt:** "Say a group of lads sit together at school and exploit our system by sending the same alert by everyone to trigger the threshold?"

- **The Threat:** A Sybil Attack. A group of pranksters (or one person with multiple tabs) artificially creating volume to trigger the "Crowd Surge" protocol.
- **Failed Idea: IP Whitelisting**
  - _Proposal:_ "School Admin adds their Gateway IP to a Trusted Allowlist."
  - _Developer Rejection:_ **"FAIL."**
  - _Reason:_ "Non-technical operators will not do this. They will ignore it. The product must be robust and easy to use."
- **Constraint:** The solution must be **Zero-Configuration**.

## Phase 5: The Final Solution (Zero-Friction Trust Engine)

We engineered a "Weighted Trust Score" system using invisible, passive signals. The Operator does nothing; the Code does everything.

### Layer 1: The "Invisible CAPTCHA" (Cloudflare Turnstile)

- **Goal:** Stop automated scripts/bots.
- **Implementation:** An invisible widget running in the background. No "click the traffic lights" puzzles (too slow for emergencies).
- **Result:** Automated spam is rejected instantly.

### Layer 2: "Device Fingerprinting" (FingerprintJS)

- **Goal:** Detect if multiple reports are coming from the same device even if they use Incognito Mode.
- **Clarification:** This is NOT biometric scanning (TouchID). It analyzes browser attributes (Screen resolution + Fonts + Battery Level + OS) to create a unique "Device Serial Number".
- **Logic:**
  - 10 Reports from 1 Device ID = Spam (Ignore).
  - 10 Reports from 10 Different Device IDs = Real Crisis (Alert).

### Layer 3: "Semantic Variance" (AI Analysis)

- **Goal:** Detect "Copy-Paste" attacks.
- **Logic:**
  - _Real Crowd:_ Uses different words ("Gun", "Shooter", "Weapon"). -> **High Trust.**
  - _Lazy Pranksters:_ Copy-paste the exact same text ("Gun in gym"). -> **Low Trust.**

### Final Approved Logic: "The Rule of Three"

We established a "Trust Threshold" (e.g., 100 Points) to trigger an alert.

| Action                      | Trust Points    | Details                                          |
| :-------------------------- | :-------------- | :----------------------------------------------- |
| **Single Anonymous Report** | **10 Points**   | Report saved; Pending Operator Verification.     |
| **Phone-Verified Report**   | **50 Points**   | High Trust (User is traceable).                  |
| **Attached Photo/Video**    | **+30 Points**  | Harder to fake in real-time.                     |
| **Crowd Boost**             | **+100 Points** | **INSTANT ALERT.** (Requires 3+ unique devices). |

### The Outcome

- **Prankster Scenario:** One kid sends a fake report. Score = 10. **Result:** Operator notified silently. Public is safe.
- **Coordinated Prank Scenario:** 3 kids send identical copy-paste reports. AI detects semantic match + IP clustering. Score penalized. **Result:** Pending Verification.
- **Real Crisis Scenario:** 3 independent students report "Gun" from different phones. Fingerprints are unique. Score > 100. **Result:** INSTANT WHATSAPP BLAST.

**Verdict:** This architecture provides **Robust Security** against pranks while ensuring **Instant Speed** for real crises, all with **Zero Configuration** for the operator.

---

**Recommendation:** For the "Zero-Friction" launch, we should stick to the Invisible Signals (FingerprintJS + Cloudflare) first. Adding SMS OTP verification costs money (SMS fees) and adds friction.

If we rely purely on FingerprintJS (Device ID) + Turnstile (Bot Check) + AI (Semantic Check), we can achieve 95% of the security without asking the user to verify a phone number at all. This keeps the "Anonymous" promise intact while stopping spam.
