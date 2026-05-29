# AK Science Dashboard — Setup Guide

This guide walks you through deploying the dashboard to GitHub Pages and connecting it to Google Sheets as a live backend.

---

## Part 1: GitHub Repo + GitHub Pages

### 1.1 Create the repo on GitHub

1. Go to https://github.com/new
2. Set **Repository name** to `ak-science-dashboard`
3. Set visibility to **Public** (required for free GitHub Pages)
4. Do **not** initialize with a README
5. Click **Create repository**

### 1.2 Push the project files

Open a terminal in the project folder and run:

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/christastic336/ak-science-dashboard.git
git push -u origin main
```

### 1.3 Enable GitHub Pages

1. In the repo, go to **Settings → Pages**
2. Under **Source**, select **GitHub Actions**
3. The `deploy.yml` workflow will run automatically on every push to `main`
4. After the first workflow run completes, your site will be live at:
   `https://christastic336.github.io/ak-science-dashboard`

---

## Part 2: Google Cloud Project + Sheets API

### 2.1 Create a Google Cloud project

1. Go to https://console.cloud.google.com
2. Click the project dropdown at the top → **New Project**
3. Name it `AK Science Dashboard` (or anything you like)
4. Click **Create**

### 2.2 Enable the Google Sheets API

1. In your new project, go to **APIs & Services → Library**
2. Search for `Google Sheets API`
3. Click on it and click **Enable**

### 2.3 Configure the OAuth consent screen

1. Go to **APIs & Services → OAuth consent screen**
2. Select **External** → **Create**
3. Fill in required fields:
   - App name: `AK Science Dashboard`
   - User support email: your Google account
   - Developer contact email: your Google account
4. Click **Save and Continue** through the remaining steps (no special scopes needed here)
5. On the **Test users** step, add `clacy@advanceky.com` as a test user
6. Click **Save and Continue**

> **Note:** While the app is in "Testing" mode, only listed test users can sign in. To allow any Google user, you would need to publish the app — but for internal KSTC use, keeping it in Testing mode with your account added is fine.

### 2.4 Create OAuth 2.0 credentials

1. Go to **APIs & Services → Credentials**
2. Click **+ Create Credentials → OAuth client ID**
3. Application type: **Web application**
4. Name: `AK Dashboard Web Client`
5. Under **Authorized JavaScript origins**, click **+ Add URI** and enter:
   ```
   https://christastic336.github.io
   ```
6. Under **Authorized redirect URIs**, click **+ Add URI** and enter:
   ```
   https://christastic336.github.io/ak-science-dashboard
   ```
7. Click **Create**
8. Copy the **Client ID** shown in the dialog (looks like `123456789-abc...apps.googleusercontent.com`)

---

## Part 3: Create the Google Sheet

### 3.1 Create a new Google Sheet

1. Go to https://sheets.google.com and create a new blank spreadsheet
2. Name it `AK Science Dashboard Data`
3. Copy the **Sheet ID** from the URL — it's the long string between `/d/` and `/edit`:
   ```
   https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID_HERE/edit
   ```

### 3.2 Create the four required tabs with exact column headers

The sheet must have exactly four tabs with the following names and headers in **row 1**.

> **Important:** Tab names and column headers must match exactly (case-sensitive).

#### Tab: `Teachers`
Columns in row 1 (A through N):
```
id | lastName | firstName | school | district | course | crpYear | program | confirmed | status | firstDayNotes | mentorRequested | teacherEmail | principalEmail
```

#### Tab: `Consultants`
Columns in row 1 (A through R):
```
id | lastName | firstName | subj1 | subj2 | address1 | address2 | homePhone | cellPhone | email | contract | launchDayBlocks | ffssBlocks | masterSeries | stipend | travelAllowance | mentorBlocks | mockExamLeader
```

#### Tab: `ConsultantNotes`
Columns in row 1 (A through C):
```
id | notes | contractOverride
```

#### Tab: `TeacherEvents`
Columns in row 1 (A through C):
```
id | eventKey | eventValue
```
*(One row per event key per teacher — e.g., one row for `t1 / fallSiteVisit_done / true`, another for `t1 / fallSiteVisit_date / 2025-09-12`)*

### 3.3 Seed the sheet with existing data (optional but recommended)

The dashboard will use the hardcoded `TEACHERS` and `CONSULTANTS` arrays as a fallback when not authenticated. To make the Sheets the source of truth, export the seed data:

1. Click **Export CSV** in the Teachers or Consultants tab
2. Paste the data into the appropriate sheet tab starting at row 2

---

## Part 4: Wire the credentials into the dashboard

Open `index.html` and find the block near the top of the last `<script>` tag labeled:

```javascript
// ═══════════════════════════════════════════════════
// GOOGLE SHEETS INTEGRATION — FILL IN THESE VALUES
// ═══════════════════════════════════════════════════
```

Replace the two placeholder strings:

```javascript
const CLIENT_ID = 'YOUR_CLIENT_ID_HERE.apps.googleusercontent.com';
const SHEET_ID  = 'YOUR_SHEET_ID_HERE';
```

Save the file, commit, and push:

```bash
git add index.html
git commit -m "Add Google credentials"
git push
```

The GitHub Actions workflow will redeploy automatically. After ~1 minute, visit:
`https://christastic336.github.io/ak-science-dashboard`

Click **Sign in with Google** in the top-right corner. After authenticating, the dashboard will load data from Google Sheets and all changes will sync automatically.

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Sign in" button does nothing | Check that the Client ID is correct and the Authorized JS origin matches exactly |
| Data doesn't load after sign-in | Check the Sheet ID and that all four tab names match exactly |
| 403 error on Sheets API | Make sure the Sheets API is enabled in your GCP project |
| Changes not saving | Check browser console for API errors; try signing out and back in |
| GitHub Pages shows 404 | Ensure Pages is set to use GitHub Actions (not a branch) and the workflow ran successfully |
