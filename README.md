![Application Command Center cover](assets/application-command-center-cover.jpg)

# Application Command Center

**Keep your job search organized, from the first saved role to the final offer.**

Application Command Center is a browser-based job application tracker built with React and Firebase. Designed around Linux, systems administration, and cloud engineering job searches, it also works for any role you want to track.

Keep company details, application progress, deadlines, follow-ups, and notes in one dashboard. Use browser-only storage to get started, or configure Firebase and Google sign-in to sync across devices.

## Features

- **Application tracking:** Add, edit, and delete applications with company, role, posting link, salary, location, dates, and notes.
- **Seven statuses:** Saved, Applied, Assessment, Interview, Offer, Rejected, and Withdrawn.
- **Dashboard and search:** View status counts, filter applications, and search company names, job titles, locations, salaries, or notes.
- **Flexible sorting:** Sort by last update, company, status, application date, deadline, or follow-up date.
- **Deadline and follow-up reminders:** See overdue, due-today, and approaching items inside the dashboard.
- **Duplicate warnings:** Review matching posting URLs and exact or similar company/title combinations before saving.
- **Activity history:** Record application creation, status changes, and completed follow-ups.
- **CSV import and export:** Back up application records and review duplicate or conflicting rows before importing.
- **Optional cloud sync:** Use Google sign-in and Cloud Firestore for real-time updates across devices.
- **Responsive dark interface:** Desktop tables and mobile-friendly application cards.

New trackers start empty. Application dates, deadlines, salaries, and other optional details stay blank until you enter them.

## Quick start

The frontend is contained in `index.html`. No npm installation or build step is required.

With Git and Python 3 installed:

```bash
git clone https://github.com/cuzzaintit321/application-command-center.git
cd application-command-center
python3 -m http.server 8000 --bind 127.0.0.1
```

Open [http://localhost:8000](http://localhost:8000) in your browser, then choose **Enter Local Demo Mode (Browser-Only)** to start without Firebase.

Press **Ctrl+C** in the terminal to stop the local server.

The page loads React, Tailwind CSS, Babel, and Firebase SDKs from CDNs, so an internet connection is needed to load these dependencies even in Local Demo Mode.

## Storage modes

| | Local Demo Mode | Firebase cloud mode |
| --- | --- | --- |
| Account | No sign-in required | Google sign-in |
| Application storage | This browser's localStorage | Cloud Firestore |
| Cross-device sync | No | Yes, using the same project and Google account |
| Setup | Choose Local Demo Mode | Configure Firebase, authentication, and rules |
| Backup | CSV export | CSV export |

Local records belong to the browser profile and site address where you entered them. Clearing browser data can remove them. Export a CSV before clearing storage, switching devices, or moving to a different site address.

Firebase configuration is also saved in the browser. Configure each new browser with the same Firebase project before signing in.

Cloud errors are displayed in the app; the app does not automatically switch to local storage when a cloud operation fails.

## Firebase setup

Firebase is optional. Complete these steps if you want Google sign-in and cloud sync.

### 1. Create a project and register a web app

In the [Firebase Console](https://console.firebase.google.com/), create or select a project and register a **Web app**. Copy its Firebase configuration values from the project settings.

See [Firebase web setup](https://firebase.google.com/docs/web/setup).

### 2. Enable Google sign-in

Open **Authentication → Sign-in method**, enable **Google**, complete the provider settings, and save.

In **Authentication → Settings → Authorized domains**, add the hostname where you run the app. Add `localhost` for local development if it is not listed. For a hosted version, add its actual hostname without a path or port.

See [Google authentication setup](https://firebase.google.com/docs/auth/web/google-signin).

### 3. Create the Firestore database

Create a **Cloud Firestore Standard edition** database using the `(default)` database ID, choose a location, and start in production mode.

The app uses the default Firestore database. See the [Firestore quickstart](https://firebase.google.com/docs/firestore/quickstart).

### 4. Publish the access rules

In the Firestore **Rules** editor, publish the following rules. They match the rules shown in the app's Firebase setup dialog:

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null
                         && request.auth.uid == userId;
    }
  }
}
```

These rules restrict client access to the signed-in user's own document subtree. User data is stored at:

| Data | Firestore path |
| --- | --- |
| Applications | `/users/{userId}/applications/{appId}` |
| Reminder preferences | `/users/{userId}/settings/preferences` |

Adding the Firebase configuration to the app does not publish these rules or enable Google sign-in. Complete both steps in Firebase before using cloud storage.

### 5. Connect the app

Choose **Configure Firebase & Security Rules**, paste your web app configuration as valid JSON, and select **Save & Connect Firebase**.

The app requires these four values:

```json
{
  "apiKey": "YOUR_FIREBASE_API_KEY",
  "authDomain": "YOUR_PROJECT_ID.firebaseapp.com",
  "projectId": "YOUR_PROJECT_ID",
  "appId": "YOUR_FIREBASE_APP_ID"
}
```

Use the actual values from your Firebase Console. Keys and string values must have double quotes, and the JSON must not contain comments or trailing commas. Additional fields from your web configuration can remain.

Select **Sign In with Google**. On another device, configure the same Firebase project and sign in with the same Google account to access the same records.

## Using the tracker

Add a job application and enter a **Company**, **Job Title**, and **Status**. Status defaults to **Saved**.

| Field | Required? | Notes |
| --- | --- | --- |
| Company | Yes | Employer or organization |
| Job Title | Yes | Position you are tracking |
| Status | Yes | One of the seven supported statuses |
| Posting URL | No | Link to the job listing |
| Salary Range and Location | No | Free-text details |
| Application Date | No | Date you submitted the application |
| Deadline | No | Last date to apply |
| Follow-up Date | No | Date you want to follow up |
| Notes | No | Recruiter details, requirements, or other context |

When an existing record changes from **Saved** to **Applied** with no application date, the app offers an optional date prompt. The date remains blank unless you save one.

### Reminder behavior

- The default warning window is **3 days** and can be adjusted from **1 to 60 days**.
- Application deadline alerts appear only for roles with **Saved** status.
- Follow-up reminders appear for **Saved**, **Applied**, **Assessment**, **Interview**, and **Offer**.
- **Rejected** and **Withdrawn** records keep their dates but do not show reminders.
- Completing a follow-up adds a history event, clears the current follow-up date, and lets you schedule another.
- Reminders use the browser's local calendar date.

Reminders appear inside the app. This version does not send email, SMS, or background push notifications.

### Duplicate review

The app checks normalized posting URLs, exact company/title matches, and similar company/title combinations. URL matching removes known tracking parameters.

Warnings help identify possible duplicates; you can open an existing record or explicitly keep a separate entry when appropriate.

## CSV backups and imports

Select **Export CSV** to download all applications in the active storage mode, including records hidden by the current search or filter.

Exported columns:

```text
id,company,jobTitle,status,postingUrl,salaryRange,location,applicationDate,deadline,followUpDate,notes,activityHistory
```

Activity history is stored as JSON inside the CSV. Quoted fields, commas, and multiline notes are supported.

To restore or import records:

1. Select **Import CSV** and choose your file.
2. Review the import preview and any duplicate warnings.
3. Choose which rows to import or skip.
4. For an existing record ID, choose **Skip (Keep Existing)**, **Replace & Deduplicate History**, or **Import as New ID**.
5. Select **Complete Import**.

For hand-prepared CSV files, use the exact column names above and `YYYY-MM-DD` dates. The `company` and `jobTitle` headers are required; rows missing either value are skipped. Missing or unsupported status values become **Saved**.

CSV exports include application fields and activity history. They do not include Firebase configuration, reminder preferences, or record-level `createdAt` and `updatedAt` timestamps. Imports assign new record timestamps, except when an explicit replacement preserves an existing creation timestamp.

### Move local records to Firebase

Local data is not uploaded automatically when you connect Firebase.

1. Export your applications while in Local Demo Mode.
2. Configure Firebase and sign in with Google.
3. Import the exported CSV and review the preview.
4. Confirm the records appear in cloud mode.

## Hosting

The app can be served by a static web host.

To use **GitHub Pages**, open the repository's **Settings → Pages**, choose **Deploy from a branch**, select **main** and **/ (root)**, and save. Use the site URL GitHub provides after deployment.

See [GitHub Pages publishing sources](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site).

For cloud mode, add the deployed site's hostname to Firebase Authentication's authorized domains. Use HTTPS for a public deployment.

## Technology

| Component | Implementation |
| --- | --- |
| Interface | React 18 and ReactDOM |
| Styling | Tailwind CSS and custom CSS |
| JSX execution | Babel Standalone in the browser |
| Authentication | Firebase Authentication with Google |
| Database | Cloud Firestore with real-time listeners |
| Local storage | Browser localStorage |
| Packaging | Single HTML frontend with CDN dependencies |

This version has no build pipeline or bundled offline assets. The frontend can be hosted statically; Firebase supplies authentication and cloud storage when configured.

## Repository files

| File | Purpose |
| --- | --- |
| `index.html` | Application UI, tracking logic, storage, and Firebase setup |
| `README.md` | Setup and usage documentation |
| `assets/application-command-center-cover.jpg` | Repository cover image |

## Troubleshooting

| Problem | What to check |
| --- | --- |
| Blank page or missing styling | Confirm the browser can load the CDN scripts and check its developer console for errors. |
| Google sign-in fails | Verify that Google is enabled and the current hostname is in Firebase's authorized domains. Allow the sign-in popup. |
| Firestore permission error | Confirm you are signed in and the rules above are published in the correct project's default database. |
| Data is missing on another device | Check the selected storage mode, Firebase project, and Google account. Local mode does not sync. |
| Firebase configuration is rejected | Use valid JSON with double-quoted keys and all four required fields. |
| A different project is still active after a configuration change | Reload the page after saving the new configuration. |
| CSV rows are missing from the preview | Check that each row has company and jobTitle values and that the headers match exactly. |
| An expected reminder is absent | Check the record's status, date, and warning window. Rejected and Withdrawn records do not show alerts. |

---

Created by [Jason Edwards](https://github.com/cuzzaintit321).
