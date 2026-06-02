# IncrediPets HRMS

In-house HR management system — employee-lifecycle system of record for IncrediPets.

**Status:** v0 clickable prototype (front-end only, no backend yet). Live: https://hrms-sable-five.vercel.app

## What it is
Replaces the scattered HR Google Sheets (recruitment sheet stays for sourcing; everything from the offer onward lives here). Spans **Offer → Onboard → Develop & Manage → Retain & Reward → Offboard & Exit**.

- **Employee portal** — own profile, salary slips, reporting line, leave.
- **HR Admin** — lifecycle board + directory (Pooja/HRBP operates; Priya owns).
- **Recruiter** — create candidate + roll out the offer letter (Module 1).

## Current scope (prototype)
- Module 1 (Offer): candidate entry → auto salary breakup (Basic 62% / HRA 31% / Special 7%) → branded offer letter → release.
- Employee self-service portal + HR lifecycle board.
- **Fake seed data only — no real PII/comp.**

## Roadmap
① Offer → ② Onboarding → ③ Directory + org chart → ④ Leave → ⑤ Develop & Manage → ⑥ Offboard → ⑦ Payslip sync.

## Build & data rules
- **Comp** lives in exactly one RLS-locked table, visible only to the comp tier; never a public field.
- Live compensation stays computed in the salary sheet (payroll calculator); HRMS stores only the gated payslip PDF.
- **No real employee data on a public URL until auth + RLS are in.**

## Stack (target)
Static front-end now → Supabase (Postgres + Auth + Storage, RLS) + Vercel.

## Deploy
Auto-deploys to the live URL on every push to `main` (Vercel ↔ GitHub git integration, connected 2026-06-03).
