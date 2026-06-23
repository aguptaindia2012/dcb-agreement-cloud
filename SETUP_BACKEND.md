# DroCon Bharat Agreement Studio — Cloud Preview: Setup

This is a **separate, optional "future version"** that adds the five things on the
roadmap: **login accounts, a shared server database, a status & approval dashboard,
role-based approvals with onboarding, and team-wide template edits.** Your current
offline app is untouched — put this in a **new GitHub repo** to test it.

It uses **Supabase** (a free Backend-as-a-Service: a real PostgreSQL database +
login + access control + audit, all in one). Nothing to install or maintain.

---

## A. Get the free database (Supabase) — ~5 minutes

1. Go to **https://supabase.com** → **Start your project** → sign in with GitHub.
2. **New project**. Pick a name (e.g. `dcb-agreements`), set a strong database
   password (save it), choose the region closest to you (e.g. Mumbai), and the
   **Free** plan. Wait ~2 minutes for it to provision.
3. In the project: **SQL Editor → New query**. Open `schema.sql` from this folder,
   paste the whole thing, and click **Run**. This creates all tables, roles,
   security rules, the audit log, and the auto-onboarding trigger.
4. **For easy testing:** Authentication → **Sign In / Providers → Email** →
   turn **OFF "Confirm email"**. (Now sign-ups log in immediately. Turn it back on
   later for production.)
5. **Project Settings → API.** Copy two values:
   - **Project URL** (looks like `https://abcd...supabase.co`)
   - **anon public** key (a long token)

> The **anon public key is safe to put in the browser** — it is meant to be public.
> The database's Row Level Security (from `schema.sql`) is what actually protects
> the data, not the key.

**Free tier is plenty for your team:** ~500 MB database, 50,000 monthly active
users for login, daily backups. No credit card required.

---

## B. Configure the app

1. In this folder, **copy `config.example.js` to `config.js`**.
2. Paste your **Project URL** and **anon public key** into `config.js`.
3. (`config.js` is the only file you edit.)

---

## C. Put it online (new GitHub repo)

Same idea as your current app — it's static files:

1. Create a **new** GitHub repository (separate from your live tool), e.g.
   `dcb-agreements-cloud`.
2. Upload these files: `index.html`, `config.js`, and (optional) `schema.sql`,
   the two `.md` guides. **Do not upload `config.example.js` is fine to include.**
3. **Settings → Pages → Deploy from branch → main → /(root)**. You get a
   `https://…github.io/dcb-agreements-cloud/` link.
4. Open the link.

> Login needs https, which GitHub Pages provides. For local testing run
> `npx serve` or `python -m http.server` in this folder and open the localhost URL.

---

## D. First run & onboarding

1. Open the app → **Create an account** with your work email + password.
   **The first account automatically becomes the `admin`.**
2. Ask each team member to create their own account. They start as **drafter**.
3. As admin, go to **Team & roles** and set each person's role:
   - **admin** — everything, manage roles.
   - **approver** — review & approve/reject agreements, edit shared templates.
   - **drafter** — create agreements and submit them for review.
   - **viewer** — read-only.

---

## E. Try the workflow

1. **New agreement** → fill title, counterparty, type, assign an approver.
   (Optionally **Import JSON** — export a draft from your offline Studio with
   "Save JSON draft" and attach it here, linking the two tools.)
2. Save as **Draft** → open it → **Submit for review**.
3. Sign in as an **approver** → **Approvals** tab → open it → **Approve** or
   **Reject** (with a reason).
4. Approver/admin can then **Mark executed** once signed.
5. Every step is recorded in **Audit log** (admin tab) and on the agreement's
   **History**.

---

## What is / isn't in this preview

**In:** real login, shared cloud storage of agreements, status workflow
(Draft → In review → Approved/Rejected → Executed), role-based permissions
enforced by the database, team onboarding, shared template store, and an audit
trail. Data is encrypted at rest (AES-256) and in transit (TLS) by Supabase.

**Not yet wired:** the full clause/letterhead **document engine** from your
offline Studio. In this preview an agreement stores its **draft JSON** (which you
can import/export to move between the offline Studio and the cloud). Merging the
document generator into the cloud app is the next build step — the data model is
already shaped for it (`agreements.data` is exactly the Studio draft object).

---

## Free database options (if you ever want alternatives)

| Option | Gives you | Notes |
|---|---|---|
| **Supabase** (recommended) | Postgres DB + Auth + access rules + storage | One free tier covers all five roadmap items; used here |
| **Firebase** | Firestore (NoSQL) + Auth | Also free; different data model (no SQL) |
| **Appwrite Cloud** | DB + Auth + storage | Free tier; similar BaaS idea |
| **PocketBase** | SQLite DB + Auth in one binary | Free, but you self-host it (e.g. on Fly.io's free tier) |
| **Neon / Turso** | Postgres / SQLite only | Free DB, but you'd add auth separately |

For your needs (small team, no ops, all five features), **Supabase is the best
fit** — which is why the preview is built on it.
