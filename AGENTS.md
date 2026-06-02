# AGENTS.md — context for AI coding tools

**Read `HANDOFF.md` first — it has the full project context, data model, roles, and the comp rule.**

Quick orientation:
- **Project:** IncrediPets HRMS — employee-lifecycle web app. Supabase (Postgres + Auth + RLS) backend; static front-end on Vercel evolving to a Supabase-Auth-gated app.
- **Tables:** `employees` (hub), `profiles` (login→role), `offers` (Module 1, holds comp).
- **Roles:** `owner`, `hr_admin`, `recruiter`, `manager`, `employee` (see `profiles.role`; helper `public.current_user_role()`).
- **🔒 Comp rule (never break):** salary lives only in `offers`, readable solely by `owner`/`hr_admin` or the releasing recruiter — enforced by RLS. Never expose comp to a role that can't see it; never store comp without RLS; never log it.
- **Secrets:** use only the **publishable** key + Project URL in the browser (see HANDOFF §5). NEVER put the `sb_secret_…` service key or DB password in client code, the repo, or this tool.
- **Workflow:** edit on a branch → PR → Jiten reviews → merge auto-deploys. Never disable RLS. Never make the repo public.
- **Next task:** wire real Supabase Auth into `index.html`, replacing the fake role-picker (see HANDOFF §6).
