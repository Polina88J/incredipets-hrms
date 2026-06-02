# IncrediPets HRMS — Handoff & Build Guide

**For:** Priya (HR architect & owner of this tool) + whatever AI builder she uses (v0, Cursor, Claude, etc.)
**From:** Jiten + Claude · **Last updated:** 2026-06-03

This one file tells you what the HRMS is, what to do first, how to build it safely, the bits your AI tool must know, and which keys to use.

---

## 0. Current state (read this first)

- **The backend is live.** A dedicated Supabase project holds the database with security rules already set up.
- **Accounts exist.** Priya = HR-Admin (full edit), Jiten = Owner.
- **The front-end is still a prototype** — it has a fake "pick a role" login. **The next build task is wiring the real login** (see §6, "Immediate next task").
- **Live URL:** https://hrms-sable-five.vercel.app
- **Code:** https://github.com/jiten-git/hrms (private)

---

## 1. What this is

The HRMS is **Priya's employee-lifecycle system of record** for IncrediPets — it replaces the scattered HR Google Sheets. It covers the employee journey **from the offer onward**:

**Offer → Onboard → Develop & Manage → Retain & Reward → Offboard & Exit.**

Recruitment (sourcing, screening) stays in the existing recruitment sheet — the HRMS picks a person up when an **offer** is made. Payroll math stays in the salary sheet; the HRMS never recomputes pay, it just shows the payslip.

**Who does what**
- **Priya** — architect & owner: decides what the tool does; HR-Admin user (full edit).
- **Pooja (HRBP)** — runs the day-to-day lifecycle once it's built.
- **Recruiter (Anushka)** — creates a candidate + rolls out the offer letter.
- **Jiten** — owner, sign-off, reviews code before it ships.
- **Yash** — keeps the tool running (maintenance), no HR data / no pay visibility.

---

## 2. What you (Priya) need to do first — access checklist

1. **Supabase login** — Jiten adds you to the Supabase project so you can see the database, tables, and users in a dashboard (no code needed). You'll log in at supabase.com.
2. **GitHub** — Jiten adds you to the repo `jiten-git/hrms` so you can edit the code (via your AI tool).
3. **Your HRMS app login** — email `priya@incredipets.in`, **temporary password from Jiten** → change it on first login. Your role is **HR-Admin** (full edit).
4. **Pick your AI builder** — recommended: **v0.dev** (it's by Vercel, where this app is hosted; you describe changes in plain English and it writes + deploys the code). Connect it to the GitHub repo above.

> You don't write code by hand. You tell your AI builder what you want; it edits the code; it ships to a link Jiten reviews.

---

## 3. How to build (the safe workflow)

1. **Describe the change** to your AI builder (e.g., "add a 'probation end date' field to the employee profile").
2. The AI edits the code on a **branch** (not directly on `main`).
3. It opens a **Pull Request** → **Jiten reviews it** → on approval it **merges and auto-deploys** to the live link.
4. Nothing reaches the live tool without Jiten's review — that's the agreed safety gate.

**Golden rule:** never paste the **secret** key or database password into your AI tool or the code. Only the *publishable* key (below) goes in the app.

---

## 4. What your AI builder must know (paste this into your AI tool's context)

> **Project:** IncrediPets HRMS — an employee-lifecycle web app. Backend: **Supabase** (Postgres + Auth + RLS). Front-end: currently a single static `index.html` on **Vercel**; evolving to a Supabase-Auth-gated app. Keep it simple and framework-light unless asked.

**Tech stack**
- Supabase (Postgres database, Auth for login, Row-Level Security for permissions, Storage for files/PDFs later).
- Front-end talks to Supabase with the **Supabase JS client** using the **Project URL + publishable key** (see §5).
- Hosted on Vercel. India region (`ap-south-1`).

**Database tables (already created)**
- `employees` — the hub. Columns: `id`, `employee_code` (e.g. IN-001), `full_name`, `designation`, `department`, `location`, `date_of_joining`, `status` (offered/accepted/active/exited), `reporting_manager_id` (→ employees.id, for the org chart), `work_email`, `auth_user_id` (→ the login), `created_at`, `updated_at`.
- `profiles` — links a login to a role. Columns: `id` (→ auth.users), `role` (one of: `owner`, `hr_admin`, `recruiter`, `manager`, `employee`), `full_name`, `employee_id`.
- `offers` — Module 1 (the offer). Columns: `id`, `employee_id`, `annual_ctc`, `basic_monthly`, `hra_monthly`, `special_monthly`, `offer_status` (offered/accepted/declined), `offer_letter_url`, `released_by`, `released_at`. **These columns hold compensation — see the comp rule below.**

**Roles & permissions (enforced in the database by RLS — do not try to bypass)**
- `owner` (Jiten) — everything.
- `hr_admin` (Priya, Pooja) — full lifecycle edit; can see comp.
- `recruiter` (Anushka) — create candidate, make offers; sees only her own candidates' comp.
- `manager` — sees own team; approves leave.
- `employee` — sees only their own profile, payslips, reporting line.
- Helper function `public.current_user_role()` returns the current user's role for use in policies.

**🔒 THE COMP RULE (most important — never break this)**
- Compensation (the `offers` CTC/salary columns) is readable **only** by `owner`/`hr_admin`, or the recruiter who released that specific offer. Employees and managers can **never** read it. This is enforced by RLS in the database.
- **Never expose any salary number to the browser for someone who isn't allowed to see it.** Never store comp in a table without RLS. Never log it.
- Payslips are stored as **gated PDF files**, not as queryable salary numbers.

**Salary breakup (when generating offers/payslips)**
- Monthly: **Basic = 62% of gross, HRA = 31% (50% of Basic), Special Allowance = 7% (the balance).**
- PF on `min(Basic, ₹15,000)`. ESI if gross ≤ ₹21,000. (Statutory math; payroll sheet remains source of truth.)

**Build order (modules, one by one)**
① Offer → ② Onboarding → ③ Directory + org chart → ④ Leave → ⑤ Develop & Manage (warnings/perf) → ⑥ Offboard → ⑦ Payslip sync.

**Hard do-NOTs**
- ❌ Never put the **secret** Supabase key (`sb_secret_…`) or the **database password** in front-end code, the repo, or this AI tool. Server-side only.
- ❌ Never disable Row-Level Security or add a table without RLS.
- ❌ Never deploy real employee data to a public URL without auth being on.
- ❌ Never make the repo public (it's HR data).

---

## 5. Credentials & API keys

**Safe to use in the app (client-side):**

| Key | Value | Use |
|---|---|---|
| Project URL | `https://dxhravqtmmxeubdwxblo.supabase.co` | Supabase client |
| Publishable key | `sb_publishable_m2rIXmKjXjV0b91EyOrIfQ_IYGsnoj3` | Supabase client (browser-safe; RLS protects data) |

Put these in a `.env.local` (see `.env.example`) or your AI tool's env settings.

**SECRET — never in the repo, the browser, or an AI tool:**
- **Secret/service key** (`sb_secret_…`) and the **database password** — server-side only (Vercel env vars / one-off admin scripts). **Get these from Jiten** (they're in his password manager / a restricted store). They are *not* in this file on purpose.

---

## 6. Useful info

**App accounts**
- `priya@incredipets.in` — HR-Admin (full edit) — temp password from Jiten.
- `jiten@incredipets.in` — Owner.

**Immediate next task (for whoever builds next — you, your AI, or Claude):**
> Wire **real Supabase Auth** into `index.html`, replacing the fake role-picker. On load: check the Supabase session; if none, show an email+password login. After login, read the user's `role` from `profiles` and show the matching view (employee / recruiter / hr_admin / owner). Read/write `employees` and `offers` through the Supabase JS client (RLS already enforces who can see what). Keep comp out of any view the role isn't allowed to see. Then redeploy.

**Links**
- Live app: https://hrms-sable-five.vercel.app
- Repo: https://github.com/jiten-git/hrms
- Supabase dashboard: https://supabase.com/dashboard/project/dxhravqtmmxeubdwxblo
- Full design spec (data flow, journey, roles): ask Jiten for the "HRMS process & access spec".

**Questions / blockers:** Jiten.
