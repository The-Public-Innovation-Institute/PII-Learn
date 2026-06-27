# PII Learn — Setup Guide

## Step 1 — Run the database schema

In your Supabase project, go to **SQL Editor** and paste the entire contents of `supabase-schema.sql`, then click **Run**.

---

## Step 2 — Create the storage bucket

In Supabase, go to **Storage → New Bucket**:
- Name: `course-files`
- Toggle **Public bucket**: on
- Click **Save**

---

## Step 3 — Create your admin account

In Supabase, go to **Authentication → Users → Add User → Create New User**:
- Email: `khahlil@thepii.org`
- Password: `#ThePII#2026`
- Click **Create User**

Then go back to **SQL Editor** and run:
```sql
UPDATE public.profiles
SET role = 'admin', full_name = 'Khahlil Louisy'
WHERE email = 'khahlil@thepii.org';
```

---

## Step 4 — Get your API credentials

**Project URL:**
Go to **Integrations → Data API** in the left sidebar. Copy the API URL shown (e.g. `https://adtgtzblbneqedkysurk.supabase.co/rest/v1/`). Your Project URL is that address without the `/rest/v1/` at the end:
```
https://adtgtzblbneqedkysurk.supabase.co
```

**Anon Key:**
Go to **Settings → API Keys**. Under **Publishable key**, click the copy icon next to the `sb_publishable_...` key.

---

## Step 5 — Add your credentials to both HTML files

Open `index.html` and `dashboard.html`. In both files, find these two lines near the top of the `<script type="module">` block and replace the placeholder values:

```js
var SUPABASE_URL      = 'https://adtgtzblbneqedkysurk.supabase.co';  // already set
var SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';                     // paste your publishable key here
```

The URL is already filled in. You only need to replace `YOUR_SUPABASE_ANON_KEY` in both files.

---

## Step 6 — Configure Supabase Auth URLs

In Supabase, go to **Authentication → URL Configuration**:
- **Site URL**: `https://YOUR_ORG.github.io/pii-learn/`
- **Redirect URLs**: add `https://YOUR_ORG.github.io/pii-learn/dashboard.html`

This tells Supabase which URLs are allowed to authenticate. Without this, sign-in will be blocked.

---

## Step 7 — Push to GitHub Pages

```bash
git init
git add .
git commit -m "PII Learn v1"
git remote add origin https://github.com/YOUR_ORG/pii-learn.git
git push -u origin main
```

Then in GitHub → **Settings → Pages**:
- Source: `main` branch, `/ (root)` folder
- Click **Save**

Your site will be live at: `https://YOUR_ORG.github.io/pii-learn/`

---

## Step 8 — Log in and test

Go to your live URL and sign in with:
- Email: `khahlil@thepii.org`
- Password: `#ThePII#2026`

You should land on the Admin dashboard.

---

## Adding instructors and students

As admin, click **Users & Roles → Add User** in the dashboard, or go to **Supabase → Authentication → Users → Add User** and then update their role in the `profiles` table:

```sql
UPDATE public.profiles
SET role = 'instructor', full_name = 'First Last'
WHERE email = 'instructor@example.org';
```

---

## Custom domain (optional)

To serve from `learn.thepii.org`:
1. GitHub → Settings → Pages → Custom Domain → enter `learn.thepii.org`
2. In your DNS, add a CNAME record: `learn` → `YOUR_ORG.github.io`
3. Update the Supabase Auth URLs (Step 6) to `https://learn.thepii.org`


---

## Deploying the Edge Function (required for Add User)

The "Add User" button in the admin dashboard requires a Supabase Edge Function. Deploy it once with the Supabase CLI:

### a. Install Supabase CLI
```bash
npm install -g supabase
```

### b. Link your project
```bash
supabase login
supabase link --project-ref adtgtzblbneqedkysurk
```

### c. Deploy the function
```bash
supabase functions deploy create-user
```

### d. Set the service role secret
In Supabase → **Settings → Edge Functions**, add this secret:
- Key: `SUPABASE_SERVICE_ROLE_KEY`
- Value: your secret key from Settings → API Keys → Secret keys

That's it. The Add User button in the dashboard will now create real Supabase auth accounts and set the correct role automatically.
