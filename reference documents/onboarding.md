# SafeCast Developer Onboarding FAQ

Welcome to the SafeCast project! This document answers common questions that have come up during development and should help you get up to speed quickly.

---

## 1. Understanding the Dashboard Metrics

### Q: The "Total Incidents" and "High Risk Alerts" cards seem to be working. How are they calculated?

**A:** These numbers are pulled directly from the database and reflect live data.

*   **Total Incidents**: This is a simple `COUNT(*)` of all rows in the `incidents` table. It shows the total number of reports ever submitted.
*   **High Risk Alerts**: This is a `COUNT(*)` of all incidents where the `severity_score` is **greater than 70**.

The backend endpoint `/api/dashboard/stats` in `backend/server.js` handles these queries.

### Q: How do we trigger a "High Risk Alert" for testing?

**A:** The current system for assigning a severity score is very basic. When a new incident is created, the backend checks the `description` field.

*   If the description contains the word **`danger`** or **`peligro`**, the incident is assigned a `severity_score` of **80**.
*   Otherwise, it gets a default score of **50**.

To test, simply submit a report with the word "danger" in the description. The "High Risk Alerts" count on the dashboard should increase by one.

### Q: What about the "Avg. Response" card? It shows "2.4h". Is that live data?

**A:** No, this is currently **placeholder (mock) data**.

*   The value `2.4` is hardcoded in the `/api/dashboard/stats` endpoint.
*   The text `"-12% vs LAST WEEK"` is hardcoded directly in the frontend component `pages/Dashboard.tsx`.

**To make this a live metric**, we need to implement a way for an "Operator" to "Acknowledge" an incident. The response time would be the difference between the incident's creation time and its acknowledgement time. This functionality is not yet built.

---

## 2. Understanding User Roles

### Q: I signed up and the app calls me an "Operator". What does that mean? Are there other user types?

**A:** Yes, the application is designed for two distinct roles: the **Public (Reporters)** and **Operators (Managers)**.

### Q: Who are the "Public" and do they need an account?

*   **Who they are:** The Public are the members of a community (e.g., students, employees, residents). They are the ones who witness and report safety concerns.
*   **Do they need an account?** **No.** A core requirement of the project is to allow for anonymous reporting to encourage people to come forward. The "Submit a Secure Report" page is accessible to anyone without needing to log in.

### Q: Who are "Operators" and why do they need an account?

*   **Who they are:** Operators are the trusted individuals responsible for managing a "Safety Circle" (e.g., an HR manager, a school principal).
*   **Why they need an account:** The sign-up and login system is exclusively for Operators. An account is required to access the **Dashboard**, which contains the confidential list of all reported incidents and analytical data for their specific circle. This ensures that sensitive information is protected and only seen by authorized personnel.

### Simple Analogy:

*   **The Public** is like a customer leaving a note in a suggestion box. They don't need to give their name.
*   **The Operator** is the store manager who has the key to the suggestion box and is responsible for reading the notes and taking action.

---

## 3. Understanding Safety Circles

### Q: What is the "Safety Circles" page? Why must it be private?

**A:** The **Safety Circles** page is the **Headquarters** or **Control Panel** for Operators. It is not for the public, and it is a major security risk to expose it.

*   **What is a Safety Circle?** A Safety Circle is a private, independent group created for a specific community. For example, "Central High Parents" is one circle, and "Downtown Service Workers" is a completely separate circle. Each circle has its own members (Operators), its own dashboard, and its own incident data.

*   **Why is it private?** This page holds the keys to the kingdom for an Operator. It must be protected behind a login for two critical reasons:

    1.  **Creating New Circles is a High-Trust Action:** The "Initialize New Circle" button allows an Operator to establish a new, official safety program for an entire organization. This is a power that cannot be given to the public, as it could be used to create fake or malicious groups.

    2.  **It is the Central Hub for Managers:** For an Operator like "Dr. Dury," this page is where they will eventually manage all their responsibilities. From here, they can see a list of all the circles they belong to, access the specific dashboard for each one, and invite other team members to join their circles. Exposing this page would be like leaving the blueprints and master keys for the entire system on the front door.

This is why access to both the **Dashboard** and the **Safety Circles** pages must be restricted to logged-in Operators only.

---
