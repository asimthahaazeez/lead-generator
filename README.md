## Lead Capture Form – Supabase Integration

The lead capture form now saves submitted lead data directly to the Supabase database. It captures the lead’s **name**, **email**, **industry**, **session_id**, and **submission timestamp**.

### How It Works

- On form submission, input data is validated.
- If valid, the lead data is inserted into the `leads` table in Supabase.
- A confirmation email is sent via a Supabase Edge Function: send-confirmation.
- The form resets and displays a success message and a queue count in the current session.

### Fixes and Code Changes

#### 1. Saving Leads to Supabase Database
- Added logic to insert lead data into the `leads` table in Supabase.
- Ensures leads are properly stored on each submission.
- Any insertion errors are logged and prevent the confirmation step.

#### 2. Removed Duplicate Confirmation Email Call
- Previously, the `send-confirmation` Edge Function was triggered twice.
- Removed the duplicate call to avoid sending redundant emails.
- The function now runs only once after a successful database insert.

#### 3. Fixed logical error in generatePersonalisedConetent
- Cannot read properties of undefined (reading '1') at line 34 in supabase/functions/send-confirmation/index.ts.
- changes:
<pre> <code> const content = data.choices[0].message?.content;  // changed from 1 to 0 </code> </pre>

#### 4. Implement session ID tracking in leads form
- Generate a UUID session ID
- Store the session ID in localStorage for persistence.
- Modify the lead insertion to include the session_id.
- On component mount and after each successful submission, query the database for all leads with the current session_id and update the local leads state.
- Update the "You're #{leads.length} in this session" message to reflect the count of leads with the current session_id from the database.
- This will ensure the session_id is stored in the database and the count reflects the actual leads submitted in the current session.
- Also removed storage key from client.ts (storage: localStorage)

### Supabase Setup

Ensure your Supabase project includes a `leads` table with the following columns:

| Column        | Type                     |
|---------------|--------------------------|
| `id`          | UUID or serial PRIMARY KEY |
| `name`        | text                     |
| `email`       | text                     |
| `industry`    | text                     |
| `session_id`  | text                     |
| `submitted_at`| timestamp                |

### 📌 Notes

- All errors during saving or email sending are logged to the browser console.
- You can update the table name or field structure in `LeadCaptureForm.tsx` as needed.
