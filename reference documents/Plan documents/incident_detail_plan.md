# Incident Details Page - Feature Roadmap

This document outlines the strategic plan for enhancing the Incident Details page. The goal is to evolve it from a simple data viewer into a comprehensive operational workspace for incident responders, aligned with the project's core principles of privacy, efficiency, and clarity.

## Operator Needs & Core Objectives

The design is driven by three primary operator needs:
1.  **Understand:** Quickly grasp the what, where, and how severe of an incident.
2.  **Act:** Take immediate, correct action and notify the right people.
3.  **Document:** Maintain a clear, timestamped record of all actions taken.

---

## Feature Tiers

### Tier 1: Must-Have Features for Core Workflow

These are foundational features for the immediate next version.

**1. Redesigned Media Gallery**
*   **Problem:** The current page only shows one image, but users can upload up to 10 media files of various types.
*   **Solution:** Implement a flexible media gallery component.
    *   Display a primary, larger media item (image or video with controls).
    *   Show thumbnails for all other attached media in a scrollable row or grid below.
    *   Clicking a thumbnail will make it the primary view.
    *   For non-visual files (e.g., audio, PDFs), display a clear file type icon and filename.
*   **Impact:** Aligns the page with actual system capabilities, provides a much more professional and useful interface, and allows operators to see all evidence.

**2. Operator Log / Internal Notes**
*   **Problem:** There is no way for operators to document the actions they take, which is a critical gap for accountability and team collaboration.
*   **Solution:** Create a private, timestamped, append-only log for operators.
    *   Automatically log key system actions like "Incident Acknowledged" or "Incident Resolved".
    *   Provide a simple input field for operators to add custom, free-text notes (e.g., "Notified on-site security via phone.").
*   **Impact:** This is the most critical feature for creating an audit trail, enabling shift hand-offs, and ensuring a defensible record of the incident response.

**3. Refined Actions Panel**
*   **Problem:** The current large "Acknowledge" button is inflexible, takes up too much screen real estate, and doesn't scale for future actions.
*   **Solution:** Replace it with a compact, state-driven panel of smaller, well-defined action buttons.
    *   **If `Pending`:** Show `Acknowledge`.
    *   **If `Acknowledged`:** Show `Resolve / Close Incident` and `Add Note`.
    *   **Always Available:** `Archive`, `Share / Escalate`.
*   **Impact:** Improves UI/UX, declutters the interface, and creates a scalable and intuitive command pattern for operators.

---

### Tier 2: Features for Enhanced Context & Collaboration

These features provide deeper insight and connect the incident to the broader SafeCast ecosystem.

**4. Contextual Map View**
*   **Problem:** The text-based location can sometimes be ambiguous. Operators need geographic context to assess the situation.
*   **Solution:** Add a "Show on Map" button.
    *   This will open a modal that uses a mapping service (e.g., Google Maps, Mapbox) to display the *general area* based on the location text.
    *   **Crucially**, it will not use or store precise GPS coordinates, adhering to the project's privacy-first architecture. A disclaimer like "Showing approximate area based on report description" will be included to manage expectations and reinforce privacy.
*   **Impact:** Provides valuable context for operators without compromising user privacy.

**5. Alert Status Module**
*   **Problem:** Operators have no visibility into whether the system's automated WhatsApp alerts were successfully triggered for high-severity incidents.
*   **Solution:** Create a small, read-only module displaying the status of automated alerts.
    *   Example: "High-Severity WhatsApp Alert Sent at Nov 25, 10:59:01 to 54 subscribers in 'Downtown Watch' Circle."
*   **Impact:** Provides critical feedback, closes the loop for the operator, and confirms the system is working as intended, directly supporting the core WhatsApp integration goals.

---

### Tier 3: Advanced Lifecycle & Integrations

These are features for long-term management and external communication.

**6. Share / Escalate Feature**
*   **Problem:** Operators may need to forward incident details to parties outside the SafeCast system (e.g., law enforcement, building management, HR).
*   **Solution:** An "Escalate" or "Share" button that opens a modal.
    *   The modal allows the operator to input an email address or select a pre-configured contact.
    *   It will include privacy-centric checkboxes to give the operator granular control over what information is sent: `[x] Include Description`, `[ ] Include Reporter Details (if available)`, `[x] Include Media Links`.
*   **Impact:** Provides a formal, trackable method for external communication while maintaining strict control over sensitive data, reinforcing the platform's privacy-first stance.

## Proposed New Layout Concept

This layout organizes information based on the operator's workflow: Understand -> Act -> Document.

| LEFT COLUMN (70%)                                                              | RIGHT COLUMN (30%)                                                              |
| :----------------------------------------------------------------------------- | :------------------------------------------------------------------------------ |
| **Incident Header**<br>`ID: inc-176395...`<br>`Circle: Downtown Watch`           | **Actions Panel**<br>`[ Acknowledge ]` `[ Add Note ]`<br>`[ Escalate ]` `[ Archive ]`   |
| ---                                                                            | ---                                                                             |
| **Primary Details**<br>`Description: ...`<br>`Location: ...` `[Show on Map]`      | **Status & Alerting**<br>`Severity: Critical (92)`<br>`Status: Pending`<br>`Alerts: WhatsApp sent.` |
| ---                                                                            | ---                                                                             |
| **Media Gallery**<br>*(Primary media viewer with thumbnails below)*            | **Operator Log**<br>`11:05 - Acknowledged.`<br>`11:06 - Notified security.`<br>`...`    |
| ---                                                                            |                                                                                 |
| **Related Incidents (Optional)**<br>*(Links to similar incidents)*             |                                                                                 |
