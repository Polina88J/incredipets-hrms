# IncrediPets HRMS

In-house HR management system — employee-lifecycle system of record for IncrediPets.

**Status:** Live and in use. Full app with a Supabase backend behind employee login (auth + RLS).
**Live URL:** https://polina88j.github.io/incredipets-hrms/  *(GitHub Pages)*

## What it is
Replaces the scattered HR Google Sheets (recruitment sheet stays for sourcing; everything from the offer onward lives here). Spans **Offer → Onboard → Develop & Manage → Retain & Reward → Offboard & Exit**.

- **Employee portal** — own profile, salary slips, reporting line, leave, documents.
- **HR Admin** (HRGM / HRBP) — lifecycle board, directory, payroll, org chart, approvals.
- **Recruiter** (HRTA) — create candidate + roll out the offer letter.

## Hosting & deploy
- **Live on GitHub Pages**, served from `Polina88J/incredipets-hrms` (People-Ops account) — free, independent of any paid host. Deploy = push `index.html` to that repo's `main`.
- **This repo (`jiten-git/hrms`) is the canonical source.** Changes land here via **pull request** (merge to `main`).
- **Vercel deployment retired (2026-06-09).** The old `hrms-sable-five.vercel.app` URL is no longer used (plan downgraded); GitHub Pages replaces it with **no functional impact** — the code lives in Git, the data lives in Supabase, neither depended on Vercel.

## Stack
Static single-file front-end (`index.html`) → **Supabase** (Postgres + Auth via SECURITY-DEFINER RPCs, RLS-locked) + GitHub Pages hosting.

## Login model
- Employees sign in with their **employee number**; first login is a **one-time password** that forces them to set their own.
- Three admin role logins (**HRGM / HRBP / HRTA**) use fixed passwords.

## Data & security rules
- Real employee + payroll data, **behind login**. Auth + RLS are in place.
- **Comp + PII are gated by role in the RPC layer** — employees see only their own pay; payslip download returns only the downloader's own row; recruiters/managers never see the company comp master.
- Payroll figures are loaded **verbatim from the Salary Sheet (Payroll Master)** — never recomputed in the app.
