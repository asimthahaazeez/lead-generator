## Overview
This document outlines the major bugs that were discovered and resolved in the Lead Capture Form integration with Supabase.

## Critical Fixes Implemented

### 1. Lead Data Not Saving to Supabase
**File**: `LeadCaptureForm.tsx`<br>
**Severity**: High<br>
**Status**: Fixed

#### Problem
Form submissions were not persisting lead data to the Supabase database. As a result, leads were not recorded, affecting marketing and follow-up workflows.

#### Root Cause
The insertion logic for the `leads` table was missing or incorrectly configured.

#### Fix
Added logic to validate form inputs and insert lead data (name, email, industry, session_id, submitted_at) into Supabase’s `leads` table.

```ts
await supabase.from('leads').insert([{
  formData.name,
  formData.email,
  formData.industry,
  formData.session_id
}]);
```


### 2. Removed Duplicate Confirmation Email Call
**File**: `LeadCaptureForm.tsx`<br>
**Severity**: Medium<br>
**Status**: Fixed

#### Problem
Users were receiving two confirmation emails per form submission.

#### Root Cause
The Edge Function send-confirmation was being called twice due to duplicate function triggers.

#### Fix
Removed the redundant call and ensured send-confirmation is triggered only after successful database insertion.

### 3.Crash in Edge Function: Cannot Read Properties of Undefined
**File**: `supabase/functions/send-confirmation/index.ts`<br>
**Severity**: High<br>
**Status**: Fixed

#### Problem
The Edge Function crashed with Cannot read properties of undefined (reading '1').

#### Root Cause
The Edge Function send-confirmation was being called twice due to duplicate function triggers.The code was incorrectly trying to access data.choices[1] when only one choice was returned from the API.

#### Fix
Corrected code to access to safely get the first choice's content.

```ts
const content = data.choices[0].message?.content;
```

### 4.Implemented Session ID Tracking in Leads Form
**File**: `LeadCaptureForm.tsx`<br>
**Severity**: Medium<br>
**Status**: Fixed


#### Problem
No session tracking for leads; the count of leads submitted during the current session was inaccurate or unavailable.

#### Root Cause
Session ID was not generated or persisted; leads were not queried by session.

#### Fix
- Generate a UUID session ID on form mount and persist it in localStorage.
- Include session_id in lead data inserted into the database.
- After each successful submission, query Supabase for all leads with the current session ID and update local state.
- Display a dynamic message showing the actual count of leads submitted during the current session, e.g., “You’re #3 in this session.”
- Removed localStorage usage from client.ts to avoid conflicts.

### Impact
- Accurate tracking of session-specific lead submissions
- Better user feedback on lead queue position in current session
- Persistent session data across page reloads

## Supabase Setup

Ensure your Supabase project includes a `leads` table with the following columns:

| Column        | Type                     |
|---------------|--------------------------|
| `id`          | UUID or serial PRIMARY KEY |
| `name`        | text                     |
| `email`       | text                     |
| `industry`    | text                     |
| `session_id`  | text                     |
| `submitted_at`| timestamp                |

### Notes

- All errors during saving or email sending are logged to the browser console.
- You can update the table name or field structure in `LeadCaptureForm.tsx` as needed.
- The confirmation email is sent via a Supabase Edge Function named `send-confirmation`.


## Post-Cloning Setup Instructions

After cloning the repository, follow these steps to set up and verify the project:

1. **Connect to a Supabase Project**
   Link the project to your personal Supabase instance.

2. **Install Supabase CLI**
   Ensure the Supabase CLI is installed on your system.
   [Installation Guide](https://supabase.com/docs/guides/cli)

3. **Run Database Migrations**
   Apply all pending migrations to set up the database schema.

4. **Deploy Edge Function**
   Deploy the required Edge Function(s) using the Supabase CLI.

5. **Check Edge Function Logs**
   Use the CLI or Supabase dashboard to monitor error logs for the deployed functions.

6. **Fix Lead Saving Issue**
   Initially, leads were not being saved to the database due to a bug, which has since been resolved.

7. **Session Tracking for Leads**
   Session ID was not being generated or persisted; leads were not queried by session.

---

# Welcome to your Lovable project

## Project info

**URL**: https://lovable.dev/projects/94b52f1d-10a5-4e88-9a9c-5c12cf45d83a

## How can I edit this code?

There are several ways of editing your application.

**Use Lovable**

Simply visit the [Lovable Project](https://lovable.dev/projects/94b52f1d-10a5-4e88-9a9c-5c12cf45d83a) and start prompting.

Changes made via Lovable will be committed automatically to this repo.

**Use your preferred IDE**

If you want to work locally using your own IDE, you can clone this repo and push changes. Pushed changes will also be reflected in Lovable.

The only requirement is having Node.js & npm installed - [install with nvm](https://github.com/nvm-sh/nvm#installing-and-updating)

Follow these steps:

sh
# Step 1: Clone the repository using the project's Git URL.
git clone <YOUR_GIT_URL>

# Step 2: Navigate to the project directory.
cd <YOUR_PROJECT_NAME>

# Step 3: Install the necessary dependencies.
npm i

# Step 4: Start the development server with auto-reloading and an instant preview.
npm run dev

**Edit a file directly in GitHub**

Navigate to the desired file(s).
Click the "Edit" button (pencil icon) at the top right of the file view.
Make your changes and commit the changes.

**Use GitHub Codespaces**

Navigate to the main page of your repository.
Click on the "Code" button (green button) near the top right.
Select the "Codespaces" tab.
Click on "New codespace" to launch a new Codespace environment.
Edit files directly within the Codespace and commit and push your changes once you're done.

## What technologies are used for this project?

This project is built with:

Vite
TypeScript
React
shadcn-ui
Tailwind CSS

## How can I deploy this project?

Simply open [Lovable](https://lovable.dev/projects/94b52f1d-10a5-4e88-9a9c-5c12cf45d83a) and click on Share -> Publish.

## Can I connect a custom domain to my Lovable project?

Yes, you can!

To connect a domain, navigate to Project > Settings > Domains and click Connect Domain.

Read more here: [Setting up a custom domain](https://docs.lovable.dev/tips-tricks/custom-domain#step-by-step-guide)