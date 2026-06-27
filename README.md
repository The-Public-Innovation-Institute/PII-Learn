# PII Learn — Learning Management Platform
**The Public Innovation Institute**

A full-stack LMS hosted on GitHub Pages with Supabase authentication, role-based dashboards, file uploads, assignments with timed windows, grading, and course content management.

---

## File Structure

```
pii-learn/
├── index.html           ← Landing page + login
├── dashboard.html       ← All role dashboards (admin / instructor / student)
├── supabase-schema.sql  ← Database schema + RLS policies
└── README.md
```

---

## 1. Supabase Setup

### a. Create your Supabase project
Go to [https://supabase.com](https://supabase.com) → New Project.

### b. Run the schema
In the Supabase **SQL Editor**, paste and run `supabase-schema.sql` in full.

### c. Create the storage bucket
Go to **Storage → New Bucket**:
- Name: `course-files`
- Public: ✅ (or private if you want signed URLs — update `getPublicUrl` calls accordingly)

### d. Create the admin user
Go to **Authentication → Users → Invite User**:
- Email: `khahlil@thepii.org`
- Password: `#ThePII#2026`

Then in **SQL Editor**, run:
```sql
update public.profiles
set role = 'admin', full_name = 'Khahlil Louisy'
where email = 'khahlil@thepii.org';
```

### e. Get your credentials
Go to **Settings → API**:
- Copy **Project URL** → replace `YOUR_SUPABASE_URL` in both HTML files
- Copy **anon/public key** → replace `YOUR_SUPABASE_ANON_KEY` in both HTML files

Replace in **both** `index.html` and `dashboard.html`:
```js
const SUPABASE_URL      = 'https://YOUR_SUPABASE_URL.supabase.co';
const SUPABASE_ANON_KEY = 'YOUR_SUPABASE_ANON_KEY';
```

---

## 2. GitHub Pages Deployment

```bash
# Create repo on GitHub (e.g. pii-learn), then:
git init
git add .
git commit -m "Initial PII Learn build"
git remote add origin https://github.com/YOUR_ORG/pii-learn.git
git push -u origin main
```

Then in GitHub → **Settings → Pages**:
- Source: `main` branch, `/ (root)` folder
- Your site will be live at: `https://YOUR_ORG.github.io/pii-learn/`

### Add your GitHub Pages URL to Supabase Auth
Go to **Supabase → Authentication → URL Configuration**:
- **Site URL**: `https://YOUR_ORG.github.io/pii-learn/`
- **Redirect URLs**: add `https://YOUR_ORG.github.io/pii-learn/dashboard.html`

---

## 3. Role System

| Role | Capabilities |
|------|-------------|
| **Admin** | Full access: manage users, view all grades, create/delete courses, manage all content and assignments, analytics |
| **Instructor** | Create courses, upload content, create timed assignments, receive and grade submissions, view gradebook |
| **Student** | View enrolled courses, access content, submit assignments within open windows, view own grades and feedback |

---

## 4. Adding Users

### Students / Instructors
As admin, click **Users & Roles → Add User** in the dashboard.

For full invite capability (sending emails), deploy a Supabase Edge Function with your `service_role` key:

```js
// supabase/functions/invite-user/index.ts
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

Deno.serve(async (req) => {
  const { email, role, full_name, password } = await req.json()
  const sb = createClient(
    Deno.env.get('SUPABASE_URL')!,
    Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
  )
  const { data, error } = await sb.auth.admin.createUser({
    email, password,
    email_confirm: true,
    user_metadata: { full_name, role }
  })
  if (error) return new Response(JSON.stringify({ error }), { status: 400 })
  await sb.from('profiles').update({ role, full_name }).eq('id', data.user.id)
  return new Response(JSON.stringify({ success: true }), { status: 200 })
})
```

Then call this function from the dashboard's `addUser()` function.

### Enrolling students in courses
In **Supabase → SQL Editor**:
```sql
insert into public.enrollments (course_id, student_id)
values ('COURSE_UUID', 'STUDENT_UUID');
```
Or build an enrollment UI in the admin Courses view (next iteration).

---

## 5. Assignment Time Windows

When creating an assignment, instructors set:
- **Opens At** — datetime when students can start submitting
- **Closes At** — datetime when submissions are no longer accepted

The dashboard enforces this client-side (status badge: Scheduled / Open / Closed). For hard server-side enforcement, add a Postgres policy:

```sql
create policy "Submissions only accepted during open window"
  on public.submissions for insert
  with check (
    exists (
      select 1 from public.assignments a
      where a.id = assignment_id
        and now() between a.opens_at and a.closes_at
    )
  );
```

---

## 6. Database Tables

| Table | Purpose |
|-------|---------|
| `profiles` | User accounts + roles |
| `courses` | Course records with instructor FK |
| `enrollments` | Student ↔ course many-to-many |
| `course_content` | Videos, audio, docs, links per course |
| `assignments` | Timed assignments per course |
| `submissions` | Student submissions + grades + feedback |

---

## 7. Custom Domain (optional)

To serve from `learn.thepii.org`:
1. In GitHub Pages settings → **Custom Domain** → enter `learn.thepii.org`
2. Add a CNAME record in your DNS: `learn` → `YOUR_ORG.github.io`
3. Update Supabase Auth URLs to `https://learn.thepii.org`

---

*Built for The Public Innovation Institute · Boston, MA*
