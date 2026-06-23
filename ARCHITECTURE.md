# DroCon Bharat Agreement Studio — Cloud Architecture & Build Plan

## The one prerequisite
All five roadmap items (login, shared store of in-progress/past agreements, a
status-and-approval dashboard, role-based approvals with onboarding, and
team-wide permanent template edits) need the same thing: **a backend = server +
database + authentication.** Rather than build and run servers, we use a managed
**Backend-as-a-Service (Supabase)** so the "server" is fully hosted and free to
start, while the front-end stays the same kind of static site you already deploy
on GitHub Pages.

```
   Browser (your existing front-end, static on GitHub Pages)
        │   supabase-js (HTTPS)
        ▼
   Supabase project
     ├─ Auth            → login accounts, sessions
     ├─ PostgreSQL DB   → agreements, profiles/roles, templates, audit_log
     │     └─ Row Level Security → access control enforced IN the database
     └─ (Storage)       → optional: store generated PDFs/DOCX later
```

Why this shape:
- **Keeps your front-end.** No framework rewrite; we add a thin data layer.
- **Security lives server-side.** Row Level Security (RLS) means even if someone
  has the app's public key, the database still refuses anything their role
  isn't allowed to do. This is the "encryption-at-rest + access control + audit"
  layer you wanted, without you running infrastructure.
- **Free to test, cheap to scale.**

## Data model (see `schema.sql`)
- **profiles** — one per user, with a `role` (admin / approver / drafter / viewer).
  Auto-created on signup; the first user becomes admin.
- **agreements** — the shared store. Holds metadata + `status`
  (draft → in_review → approved/rejected → executed) + `data` (the full Studio
  draft JSON) + owner + assigned approver.
- **template_overrides** — team-wide standard clauses per template (the server
  version of today's local "Edit clause templates").
- **audit_log** — who did what, when (created/submitted/approved/rejected/
  executed/role_changed/template_saved).

## How it maps to the five items
1. **Login accounts** → Supabase Auth + `profiles`.
2. **Server store of in-progress/past agreements** → `agreements` table; the app
   reads/writes it instead of (or alongside) localStorage.
3. **Status & approval dashboard** → the **Agreements** and **Approvals** tabs,
   driven by `agreements.status`.
4. **Role-based approvals + onboarding** → `profiles.role`, the **Team & roles**
   admin screen, RLS policies, and the `admin_set_role` function.
5. **Permanent, team-wide template edits** → `template_overrides`, written by
   admins/approvers, read by everyone — the multi-user successor to the local
   template editor.

## Suggested build sequence (incremental, each independently testable)
1. **Auth + profiles** (login, first-admin bootstrap). ✅ in this preview
2. **Agreements CRUD + list** (shared store, replaces local-only). ✅
3. **Status workflow + approvals dashboard** (submit/approve/reject/execute). ✅
4. **Roles, onboarding, audit log** (Team screen, RLS, history). ✅
5. **Team-wide template store** (shared overrides). ✅ (basic; clause editor to merge)
6. **Merge the document engine** — bring the offline Studio's clause/letterhead/
   Word/PDF generation into the cloud app so an agreement is drafted, reviewed,
   approved and exported in one place. (Next major step.)
7. **Hardening** — email confirmation on, optional approval-only status
   transitions enforced by a DB trigger, generated-document storage, per-team
   data partitioning if you ever run multiple entities.

## Migration & coexistence
- Your **offline app stays as-is.** The cloud app is a separate repo/URL.
- The bridge today: export a draft from the offline Studio ("Save JSON draft")
  and **Import** it into a cloud agreement; download it back any time. Because
  `agreements.data` is exactly the Studio draft object, step 6 (merging the
  engine) needs no data reshaping.

## Cost & security notes
- Supabase **free tier**: ~500 MB Postgres, 50k monthly active auth users, daily
  backups, TLS in transit, AES-256 at rest. No card to start.
- RLS policies are in `schema.sql`; they are enforced by Postgres, so the browser
  cannot bypass them.
- The browser uses the **anon public** key only — safe to expose. Never put the
  **service_role** key in the front-end.
