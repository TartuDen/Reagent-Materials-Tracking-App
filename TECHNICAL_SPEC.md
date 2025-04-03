Technical Specification: Reagent & Materials Tracking App

1. Overview
This document outlines the requirements, architecture, and functionality for a mobile and web application that tracks laboratory reagents and materials. The primary goals are to:

    * Maintain an up-to-date inventory of reagents (location, quantity, etc.).

    * Track who takes items out, when, and how much they use.

    * Provide a simple interface on Android (via Cordova) and in web browsers on desktop.

    * Offer minimal but sufficient security via user roles and hashed passwords.

    * Potentially expand to add email notifications and other features later.



2. Goals & Features
    # Inventory Management

        * Store detailed info for each reagent (name, CAS, synonyms, etc.).

        * Track real-time location (shelf vs. taken by a user).

        * Update amounts used or remaining.

    # User Management

        * Regular users can search, view, and take out/return reagents.

        * Admins can add/edit/delete reagents and manage users (including approving, removing, or updating roles).

    # Notifications

        * In-App Notifications: Provide reminders or alerts (e.g., item not yet returned, item must be stored in fridge, near expiration, etc.).

        * (Future Enhancement: Email): Option to send email alerts for low stock or overdue returns, etc.

    # Photo & MSDS Handling

        * Store small thumbnail images of the reagent container.

        * Store or link to MSDS (likely PDF).

        * For the demo, files will be kept on the same server for simplicity.

    # Cross-Platform

        * Android mobile app built with Cordova (HTML/CSS/JS).

        * Web-based interface for desktop browsers using the same backend.

    # Security & Authentication

        * Email + Password registration.

        * Passwords stored in a hashed + salted format (bcrypt or Argon2).

        * Basic admin vs. regular user roles.

        * HTTPS to protect credentials in transit (for production/deployment).

    # Scalability

        * Up to ~3000 reagents and ~15-20 users in the future.

        * Using MongoDB (free tier) for data storage is sufficient for this scale.



3. Architecture
    3.1 High-Level Diagram
```bash
[ Cordova App ] --\
                --> [ Node.js/Express Backend ] --> [ MongoDB Atlas ]
[ Web Browser ] --/
```
    * Cordova App:  Deployed as an APK for Android. Communicates with the backend via REST APIs (HTTPS).

    * Web Browser: Uses the same REST APIs for CRUD operations, user authentication, etc.

    * Node.js/Express Backend: Hosts the business logic, user authentication, and file upload endpoints.

    * MongoDB Atlas: Stores data for reagents (matStored), user accounts (users), and transaction logs (matTaken).


4. Detailed Requirements
    4.1 User Roles & Permissions

    * Admin - Add/edit/delete reagents, manage users (approve, remove, change roles), configure notifications, view all logs in detail.

    * Regular - View reagents and details, take/return items, update the remaining amount, view limited usage logs (primarily their own).

    4.2 Data Model
```bash
matStored (Materials/Inventory)
Field	Type	Description
materialID	Number	Unique ID for the reagent.
WHcode	String	Warehouse code or internal reference.
chemName	String	Chemical/Reagent name.
synonyms	[String]	Array of synonyms for easier searching.
bruttoF	String	Molecular (brutto) formula.
CASnum	String	CAS Number.
retestExp	Date	Retest/Expiration date.
arrived	Date	Date reagent arrived.
orderAmount	Number	Original order amount (float).
remainAmount	Number	Remaining amount (float).
units	String	One of ["kg", "g", "mg", "L", "mL"].
shelf	String	Storage location (e.g. “Shelf 56”, “Kelder”).
MSDS	String	Link/path to PDF.
storageTemp	String	One of ["-20°C", "2-8°C", "15-25°C"].
image	String	URL/path to the thumbnail image (stored on backend).
inStorage	Boolean	true if physically in storage, false if a user has it.
```

```bash
users (User Accounts)

Field	Type	Description
userID	Number	Unique ID for the user.
userName	String	Full name of the user.
userEmail	String	Email (also used for login).
passwordHash	String	Hashed + salted password.
role	String	Either "admin" or "regular" (default "regular").
verified	Boolean	Indicates if email is verified (optional, for future use).
```

```bash
matTaken (Usage Log)

Field	Type	Description
materialID	Number	ID of the material that was taken.
userID	Number	ID of the user who took/returned the material.
date	Date	Date the event occurred.
time	String	Time in HH:MM format.
takenAmount	Number	Amount used (float).
actionType	String	"take" or "return". (Alternatively, separate logs if needed)
```

4.3 Core Use Cases
1. Login / Registration

  * Registration: Admin can create new user accounts or allow self-signup with email verification (future).

  * Authentication: On successful login, issue a session or JWT token for subsequent requests.

 2. Search & View Reagents

  * Users can search by name, synonym, CAS, or partial matches.

  * Results show list items with location (shelf or who has it) + short info.

 3. View Reagent Details

  * Detailed screen with:

     - Photo thumbnail

     - Shelf/storage temperature
     
     - Remaining amount, expiration date

     - Link to PDF MSDS

   * Indicate if it’s in storage or who has it.

 4. Take Out / Return

  * User “takes out” reagent:

    - inStorage set to false.

    - A log entry is created in matTaken.

    - System updates remainAmount (if the user indicates how much is used).

  * User “returns” reagent:

    - inStorage set to true.

    - A “return” log entry is created (or the same table with actionType: "return").

    - Possibly update remainAmount if the user indicates how much is left.

 5. Notifications

  * In-App:

    - Admin sets triggers (e.g., near expiration, not returned for X days).

    - Alerts appear on user’s next login or via a small notification panel within the app.

  * (Future) Email:

    - Could be added using a service like SendGrid/Mailgun for triggered or scheduled emails.

 6. Admin Features

  * Admin can add/edit/delete reagents:

    - Upload images (thumbnail) and attach PDF MSDS.

  * Admin can manage user roles.

  * Admin can view logs for all usage events.

  * Admin can configure which notifications to show and to whom.

5. Technology Stack
 1. Frontend (Mobile)

   * Cordova with HTML/CSS/JavaScript.

   * Possibly frameworks like Vue/React, but can be pure HTML/JS if it’s simpler.

    * Deployed as an APK for Android.

 2. Frontend (Web)

    * A responsive web interface using the same or similar HTML/CSS/JS files.

    * Access via any modern browser (Chrome, Firefox, Edge, etc.).

 3. Backend

    * Node.js + Express for RESTful APIs.

    * File handling (images, PDFs) via multer or similar library.

    * Send notifications or scheduled tasks via Node.js CRON if needed.

 4. Database

    * MongoDB Atlas (free tier).

    * 512 MB storage limit – sufficient for the demo.

    * Keep an eye on PDF/image sizes if storing them on the server.

 5. Authentication & Security

    * bcrypt for password hashing.

    * HTTPS recommended for production.

    * Simple role-based access (admin vs. user).


6. Implementation Plan
    1. Project Setup

        * Initialize a Git repository.

        * Create a Cordova project for the mobile app.

        * Set up a Node.js/Express backend with a basic folder structure.

        * Configure MongoDB Atlas connection.

    2. Backend Development

        * Models/Schemas for users, reagents, logs.

        * Routes/Controllers for:

            - Authentication (login/register)

            - CRUD for reagents (matStored)

            - Logging transactions (matTaken)

            - File uploads (images, PDFs)

        * Security: add role-check middleware for admin-only routes.

    3. Cordova App

        * Basic views: Login, Search, Reagent Details, Take/Return.

        * Use REST API calls to the Node.js server.

        * Display in-app notifications (maybe a simple “notifications” screen or banner).

    4. Web Interface

        * Either reuse Cordova code (via a shared codebase) or build a separate small front-end.

        * Ensure it’s responsive for desktop usage.

    5. Notifications

        * Implement a simple in-app notification system:

            - A page or panel showing notifications for the logged-in user (overdue returns, near expiration items, etc.).

            - Provide admin an interface to configure or manually trigger notifications.

    6. Testing

        * Backend: Write unit tests for routes (using Jest, Mocha, or similar).

        * Mobile: Manual testing on an Android device or emulator.

        * Web: Manual testing in Chrome/Firefox + possibly basic automation with Cypress or similar.

    7. Deployment

        * Deploy Node.js server to a free hosting platform (e.g., Render, Railway, or Heroku’s free tier if available).

        * Use free-tier MongoDB Atlas.

        * Generate an Android APK from Cordova for internal distribution.

        * Provide a URL for the web version.

7. Frontend Functional Addendum
    1. Taking Out & Returning Reagents

         * A reagent can be in one of two states: in storage (inStorage = true) or taken out (inStorage = false).

         * Take Out:

             * No immediate amount is prompted (the user might not know how much they will eventually use).

             * Creates a matTaken log entry (actionType = "take") with takenAmount = 0.

             * inStorage is set to false.

         * Register Usage (Partial Consumption):

             * While the reagent is out, the same user can log one or more “usage” events (actionType = "use"), each time specifying how much they used.

             * The system checks if the usage exceeds remainAmount. If it does, it warns the user that an admin correction might be needed.

             * This repeated usage can happen until the user returns the reagent.

         * Return:

             * If no usage has been logged since the last “take” event, the system warns: “You forgot to register any consumption. Are you sure?”

             * If the user confirms, we log a return event. The reagent goes back to inStorage = true.

    2. Notifications

        * User-Specific Notifications: A user sees only the notifications meant for them (matched by userID in the notifications array) or null for system-wide.

        * Admin Notifications: Admin can view all notifications.

        * Potential triggers for notifications in the real app could be:

            - Low stock alerts

            - Overdue returns

            - Correction requests

    3. Admin Panel

        * All Notifications: The admin sees every notification in the system, with details of date, message, and the user ID.

        * Manage Users: A simple read-only list in the demo. Real functionality could allow role changes, deletions, etc.

        * Manage Reagents: A basic listing with remain amounts. In the real app, the admin could edit remain amounts or other details from here.

    4. Data Validation

        * The UI checks if the user tries to register usage with a negative or non-numeric amount. It also warns if usage is bigger than the recorded remain.

        * A future improvement could add a “Request Correction” button that sends a special notification to the admin.

    5. Security & Roles

        * Regular Users:

            - See only their notifications.

            - Can take out or return a reagent only if it’s in storage (take out) or if they themselves have it (return).

            - Can register usage if they currently have the reagent checked out.

        * Admin Users:

            - Have full visibility of notifications, user list, and reagent list (the “Admin Panel”).

            - In the final app, admins would also handle correction requests or forcibly return items if needed.

    6. UI/UX Considerations

        * Collapsible sections are used to keep the main details screen simpler (key info at the top, advanced info and usage history hidden by default).

        * The navigation bar toggles for mobile (hamburger menu) using Bootstrap’s default components.


7. Future Enhancements
    * Email Notifications: For low stock, overdue returns, or near-expiration items.

    * Barcode/QR Scanning: Speed up check-out/in.

    * Offline Mode: If needed in environments with poor connectivity.

    * Advanced Reporting: Monthly usage stats, cost tracking, etc.

    * 2FA: Add TOTP (e.g., Google Authenticator) if security needs increase.

