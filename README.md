# Excel EasyBill — PWA

Turn the Excel sales file a business already uses into Colombian electronic
invoices through the **Factus API** — no accounting software, no API knowledge
required.

> "Upload the Excel your business already uses, review the data, and generate
> electronic invoices in Factus — without learning accounting software."

This repository contains **only the PWA (frontend)**. The backend, database,
and Factus integration live in a **separate repository**.

---

## Architecture

```
┌─────────────────┐
│      PWA        │  ◄── THIS REPO
│ React + TS      │
│ Vite + SheetJS  │
└────────┬────────┘
         │ HTTPS / JSON
         ▼
┌─────────────────┐
│    BACKEND      │  ◄── separate repo (Node.js + Express + TS)
└───────┬─────────┘
        │
   ┌────┼───────────┐
   ▼    ▼           ▼
PostgreSQL  Validator   Factus API
```

**Security boundary (non-negotiable):** the PWA never talks to Factus directly
and never holds Factus credentials or tokens. The flow is always
`PWA → Backend → Factus`. OAuth2 and token renewal are the backend's job.

---

## User flow (MVP)

```
Start → Configure Factus → Upload Excel → Detect columns →
Validate data → Preview → Confirm → Generate invoices →
(Backend → Factus) → Results → Download PDF
```

### What the PWA does

1. **Connect Factus** — collect credentials and send them to the backend.
2. **Upload Excel** — read `.xlsx` in the browser with SheetJS.
3. **Validate** — surface fixable per-row errors before anything is sent.
4. **Convert** — parse Excel into internal JSON; group rows by `factura_id`.
5. **Batch invoicing** — ask the backend to generate multiple invoices.
6. **Results** — show per-invoice status and offer PDF download.

### Excel template

Expected columns:

```
factura_id, fecha, tipo_doc, documento, cliente, email,
producto, cantidad, precio, iva, forma_pago
```

Rows that share the same `factura_id` are grouped into a single invoice with
multiple line items.

---

## Tech stack

| Concern     | Choice                                |
|-------------|---------------------------------------|
| Framework   | React                                 |
| Language    | TypeScript                            |
| Build tool  | Vite                                  |
| PWA         | Vite PWA (installable, offline shell) |
| Excel parse | SheetJS                               |
| Deploy      | Vercel                                |

---

## Scope

**In scope (MVP):** connect Factus, upload `.xlsx`, validate, convert, batch
invoicing, results + PDF download.

**Out of scope:** inventory, accounting, POS, payroll, support documents, debit/
credit notes, native mobile app, Excel add-in, Google Sheets, full
conversational AI, and any direct Factus calls from the frontend. AI column
mapping is a post-MVP differentiator.

---

## Status

Early stage — PWA scaffolding not yet started. See
[`AGENTS.md`](./AGENTS.md) for the full working guide and architecture rules,
and `Excel EasyBill.md` for the original product brief.
