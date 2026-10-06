# CPDA & Budget Fund Management Portal (PGSL prototype)

Web portal for the Accounts Section of IIT Ropar to manage CPDA claims and budget funds.
**Stack:** React (Vite) frontend, Node.js + Express backend, PostgreSQL database.

All demo people and data are fictional. Password for every demo login: `Demo@1234`

## Run it (3 steps)

Requirements: Node.js 18+ and PostgreSQL 14+.

1. Create the database (once):
   ```sql
   CREATE USER cpda WITH PASSWORD 'cpda123';
   CREATE DATABASE cpda_portal OWNER cpda;
   ```
2. Start the server (creates tables and loads demo data automatically on an empty database):
   ```bash
   cd server
   cp .env.example .env        # edit DATABASE_URL if your DB settings differ
   npm install
   npm start
   ```
3. Open **http://localhost:4000** (the built React app is already included in `client/dist`).

To change the frontend: `cd client && npm install && npm run dev` and open http://localhost:5173 (API is proxied to port 4000). Run `npm run build` there to refresh `client/dist`.

**Docker alternative:** `docker compose up --build`, then open http://localhost:4000.

Reset demo data any time: `cd server && npm run db:reset`

## Demo logins (one per role, each gets its own dashboard)

| Role | Login | What to show |
|---|---|---|
| Faculty (CSE) | anita.sharma@iitrpr.demo | Balance, ledger, new claim, claim in review, paid claim |
| Faculty (EE) | kavya.rao@iitrpr.demo | Late claim with special-approval request |
| Faculty, long leave | sanjay.gupta@iitrpr.demo | Hard rule block (cannot submit) |
| HoD (CSE / EE / ME) | hod.cse@ / hod.ee@ / hod.me@iitrpr.demo | Recommend / return / reject, department analytics |
| Accounts AR | ar@iitrpr.demo | Receive and assign, final review, mark paid, pro-rata calculator |
| Accounts junior | junior1@ / junior2@iitrpr.demo | Register entry (page, serial, budget head, amount passed) |
| Registrar | registrar@iitrpr.demo | Approval step |
| Dean FAA | dean@iitrpr.demo | Approval step |
| Director | director@iitrpr.demo | Final approval, special approvals |
| Admin | admin@iitrpr.demo | Users, rules, blocks, allocation, carry-forward, audit log |

Suggested demo: log in as `anita.sharma`, submit a claim, then log in as `hod.cse`, `ar`, `junior1`, `registrar`, `dean`, `director` in turn to approve it, and finally `ar` to mark it paid.

## Features implemented (matches the slides)

**Must have:** role-based login and dashboards - claim form for every category (conference/TA and contingency/membership) - approval routing and status tracker - faculty balance dashboard - Accounts register entry and sanction - auto ledger per block - rule checks.
**Should have:** bill upload stored in PostgreSQL (`bytea`, SHA-256 hash, real file-type check) - in-app notifications (+ email if SMTP is set) - pro-rata allocation - carry-forward capped at Rs 3 lakh - PDF claim form and Excel ledger export - special approval request to the Director.
**Could have:** department analytics - "no international conference yet" list - e-signature trail (name and time on every step) - mobile-friendly layout.
**Not in scope:** payment gateway, payroll/HR, procurement, project-grant accounting.

## Workflow (from the Accounts notes)

Faculty -> HoD -> AR (receive and assign) -> Junior staff (verify, register entry) -> AR (review) -> Registrar -> Dean FAA -> Director -> Paid.
Edit the `STEPS` array in `server/src/workflow.js` to add, remove or reorder steps (for example the "GRO" step from the handwritten notes once confirmed).

## Rules (stored as data, editable by Admin, no code change)

Rs 1 lakh per year - 3-year block - conference/TA capped at 70% of the grant - memberships max 3 per year - claim within 30 days - carry-forward max Rs 3 lakh.
Hard blocks (cannot be waived): not past probation, on leave beyond 30 days, amount above balance.
Soft rules (can be sent as a special approval request): 70% cap, late claim, no prior approval reference, not attended, membership limit, advance payment.

## How the ledger works

`ledger_entries` is append-only (a database trigger blocks UPDATE and DELETE). Balances are always calculated from it:
submit claim = commitment, final approval = commitment released + expenditure, reject / return = commitment released.

## Project layout

```
server/db/schema.sql      tables      server/db/seed.js     demo data
server/src/rules.js       pro-rata maths and settings
server/src/workflow.js    approval chain and categories
server/src/services/      claims (rules + workflow), ledger, pdf, notify
server/src/routes/        auth, claims, accounts, admin, misc (analytics, notifications)
client/src/pages/<role>/  one dashboard per role
```

## Notes and limits of this prototype

- Meant for demonstration: use HTTPS, a strong `JWT_SECRET` and real SMTP settings before any real use.
- Rule values and the exact approval chain should be confirmed with the Accounts Section.
- The 70% cap is calculated on allocations (excluding carried-in balance); confirm this reading with Accounts.
