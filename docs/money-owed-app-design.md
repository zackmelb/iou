# Money Owed / Owing App Design

## 1) Product overview
A lightweight app for tracking:
- **Money you owe** (debts)
- **Money others owe you** (receivables)

Primary goals:
- Fast entry in under 10 seconds
- Clear current balance with each person
- Gentle reminders without harassment
- Audit-friendly history of every change

## 2) Target users
- Roommates splitting bills
- Friends sharing group expenses
- Freelancers tracking small unpaid invoices
- Families managing informal loans

## 3) Core user stories
1. As a user, I can add a person/contact with name and preferred payment methods.
2. As a user, I can record an amount, direction (**I owe** vs **they owe me**), due date, and note.
3. As a user, I can make partial payments and keep a full payment history.
4. As a user, I can see net balance per person and globally.
5. As a user, I can set reminders for upcoming/overdue items.
6. As a user, I can attach proof (receipt/photo/message screenshot).
7. As a user, I can export data (CSV/PDF) for personal records.

## 4) MVP feature set
- Authentication (email + password or social sign-in)
- Contacts list
- Add debt/receivable entry
- Ledger timeline per contact
- Mark paid / partial paid
- Due-date reminders (push + email)
- Simple analytics dashboard
  - total owed by me
  - total owed to me
  - overdue totals
- CSV export

## 5) Data model (high-level)

### User
- id (UUID)
- email
- display_name
- preferred_currency
- timezone
- created_at, updated_at

### Contact
- id (UUID)
- user_id (FK)
- name
- phone/email (optional)
- payment_handle (optional)
- created_at, updated_at

### Entry
Represents a money obligation.
- id (UUID)
- user_id (FK)
- contact_id (FK)
- direction (enum: `I_OWE`, `OWED_TO_ME`)
- principal_amount (decimal)
- currency
- due_date (nullable)
- status (enum: `OPEN`, `PARTIALLY_PAID`, `PAID`, `DISPUTED`, `CANCELLED`)
- note
- created_at, updated_at

### Payment
- id (UUID)
- entry_id (FK)
- amount (decimal)
- paid_at (timestamp)
- method (cash/bank/venmo/etc.)
- note

### Reminder
- id (UUID)
- entry_id (FK)
- remind_at
- channel (push/email/sms)
- sent_at (nullable)

### Attachment
- id (UUID)
- entry_id (FK)
- file_url
- mime_type
- uploaded_at

## 6) Business rules
- Remaining balance = `principal_amount - sum(payments.amount)`
- Status auto-updates:
  - remaining <= 0 => `PAID`
  - 0 < remaining < principal => `PARTIALLY_PAID`
  - remaining == principal => `OPEN`
- Entry direction decides sign in rollups:
  - `OWED_TO_ME` adds to assets
  - `I_OWE` adds to liabilities
- Overdue = `due_date < today` and status not `PAID`

## 7) UX flow (mobile-first)
1. **Home dashboard**: cards for “You Owe”, “Owed to You”, “Overdue”.
2. **Quick Add FAB**: amount + direction + contact + due date.
3. **Contact detail screen**: net balance, timeline of entries/payments.
4. **Entry detail**: edit metadata, add payment, attach receipt, set reminder.
5. **Reports/export**: monthly summary and CSV/PDF output.

## 8) Notification strategy
- Default reminder presets: 3 days before, due day, 3 days overdue.
- Rate limits: max 1 reminder/day per entry.
- Tone templates:
  - Friendly (“Just a reminder this payment is due soon.”)
  - Firm (“This payment is overdue.”)

## 9) Suggested tech stack
- **Frontend**: React Native (Expo) or Flutter
- **Backend**: Node.js + NestJS (or FastAPI)
- **Database**: PostgreSQL
- **Auth**: Supabase Auth / Firebase Auth / Auth0
- **Storage**: S3-compatible bucket for attachments
- **Notifications**: Firebase Cloud Messaging + email provider

## 10) Security & privacy
- Encrypt data in transit (TLS) and at rest.
- Per-user data isolation in API and DB queries.
- Optional app lock (PIN/biometric).
- Minimal personal data collection.
- Export + delete account workflow for compliance.

## 11) API sketch
- `POST /contacts`
- `GET /contacts`
- `POST /entries`
- `GET /entries?status=open&direction=owed_to_me`
- `POST /entries/{id}/payments`
- `POST /entries/{id}/reminders`
- `GET /reports/summary?month=YYYY-MM`
- `GET /exports/csv`

## 12) Metrics to track
- Weekly active users
- Entries created per active user
- Percentage of entries marked paid
- Reminder open/engagement rate
- 30-day retention

## 13) Future enhancements
- Shared household/group ledger
- OCR receipt parsing to auto-fill amount/date
- Auto-reconciliation from bank statements (opt-in)
- Smart nudges powered by behavior patterns
- Multi-currency conversion with FX snapshots

## 14) 2-week MVP implementation plan
- **Week 1**
  - Project scaffolding, auth, contacts, entry CRUD
  - Payment logging and status automation
- **Week 2**
  - Dashboard rollups, reminder scheduler, CSV export
  - QA, polish, and basic analytics
