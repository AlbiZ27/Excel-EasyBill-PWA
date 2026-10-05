# AGENTS.md — Excel EasyBill (PWA)

> Guidance for AI agents and contributors working in **this repository**.
> This repo contains **only the PWA (frontend)**. The backend, database, and
> Factus integration live in a **separate repository**.

---

## 1. What this project is

**Excel EasyBill** turns a business's existing Excel sales file into Colombian
electronic invoices through the **Factus API**, without the user having to learn
an accounting system or touch an API.

The product promise:

> "Upload the Excel your business already uses, review the data, and generate
> electronic invoices in Factus — without learning accounting software."

### This repository = the PWA only

The full system has three parts. **Only the first lives here:**

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

---

## 2. Non-negotiable security rule

**The PWA MUST NEVER talk to Factus directly. Factus credentials and tokens
MUST NEVER live in or pass through the frontend.**

```
❌ PWA → Factus directly
✅ PWA → Backend → Factus
```

Factus uses OAuth2 with access tokens that expire every hour. Authentication
and token renewal are **100% the backend's job**. If a task asks the PWA to
hold a Factus token, call a Factus endpoint, or store Factus secrets, that task
is wrong — stop and flag it.

The PWA only ever talks to **our own backend** over HTTPS/JSON.

---

## 3. MVP scope — what the PWA must do

The PWA drives this exact user flow (and nothing more in v1):

```
Start → Configure Factus → Upload Excel → Detect columns →
Validate data → Preview → Confirm → Generate invoices →
(Backend → Factus) → Results → Download PDF
```

The six functional pillars (frontend responsibilities in **bold**):

1. **Connect Factus** — collect credentials in a form, send to backend. The backend handles OAuth.
2. **Upload Excel** — read `.xlsx` in the browser.
3. **Validate** — show errors before anything is sent.
4. **Convert** — read Excel into internal JSON; backend maps it to Factus shape.
5. **Batch invoicing** — trigger the backend to send multiple invoices.
6. **Results** — show per-invoice status and offer PDF download.

---

## 4. Key frontend behaviors

### Excel reading happens in the browser
Use **SheetJS** to parse `.xlsx` client-side into JSON. We do **not** need to
upload the raw Excel file to the server — only the extracted/validated data.

```
Excel → PWA → SheetJS → JSON → (send to backend)
```

### Only `.xlsx` in the MVP
No CSV, no Google Sheets in v1.

### Official template + column mapping
- Offer a **downloadable official Excel template**.
- Expected columns: `factura_id, fecha, tipo_doc, documento, cliente, email, producto, cantidad, precio, iva, forma_pago`.
- Also prepare a **column-mapping UI** (user maps their columns → system fields). AI-assisted mapping is a *later* MVP+ feature — leave the seam, don't build the AI now.

### Product grouping (important)
One row is **not** one invoice. Rows sharing the same `factura_id` group into a
single invoice with multiple `items`. The PWA must group rows by `factura_id`
before preview.

```
FAC001 → Camiseta ×2, Pantalón ×1
FAC002 → Zapatos ×1
```

### Client-side validations (run before sending)
Validate rows in the PWA and surface fixable errors per row:
- Valid date
- Document present
- Client present
- Product present
- Quantity > 0
- Price >= 0
- Valid IVA
- Valid payment method
- Invoice has at least one product

Let the user either **fix the Excel** or **continue with the valid rows only**
(the MVP allows continuing with the valid subset).

### Reference code / duplicate protection
Each invoice carries a unique internal `reference_code` (e.g. `FE-2026-000001`).
Factus uses `reference_code` to deduplicate. The PWA must **prevent accidental
double submission** (e.g. disable the "Generate" button after first click,
guard against double-generation).

### Results & PDF
Show a results table (invoice, client, status, action). For accepted invoices,
offer **PDF download** (the backend fetches it from Factus). **PDF only** in the
MVP — XML is a later iteration.

---

## 5. Tech stack (PWA)

| Concern      | Choice                          |
|--------------|---------------------------------|
| Framework    | React                           |
| Language     | TypeScript                      |
| Build tool   | Vite                            |
| PWA          | Vite PWA (installable, offline shell) |
| Excel parse  | SheetJS                         |
| Deploy       | Vercel                          |

The backend (separate repo) uses Node.js + Express + TypeScript, with
PostgreSQL and the Factus API.

---

## 6. Explicitly OUT of scope (do not build)

Do not add any of these to the PWA unless the scope changes:

❌ Inventory ❌ Accounting ❌ POS ❌ Payroll
❌ Support documents ❌ Debit notes ❌ Credit notes
❌ Native mobile app ❌ Excel add-in ❌ Google Sheets
❌ Full conversational AI ❌ Direct Factus calls from the frontend

AI column mapping is a **post-MVP differentiator** — not now.

---

## 7. Conventions for agents

- **Language of artifacts**: all code, identifiers, comments, UI copy, and docs
  default to **English** unless the user explicitly asks otherwise or the
  existing project clearly uses another language. (Note: domain field names from
  the template like `factura_id`, `cliente` are Spanish by data contract — keep
  those as the data contract defines them.)
- **Match existing patterns**: read before writing. Follow the project's
  structure, naming, and libraries rather than introducing new ones.
- **Verify before claiming**: read files and run the build/tests before saying
  something works.
- **Respect the architecture boundary**: if a change would make the PWA hold
  Factus secrets or call Factus directly, STOP and flag it — that work belongs
  in the backend.
- **Keep scope tight**: solve the task asked; don't add features from the "out
  of scope" list.

---

## 8. Project skills

Shared agent skills live in `.agent/skills/` and travel with the repo. Match by
task context and load the relevant `SKILL.md` before working.

| Skill | Path | Load when |
|-------|------|-----------|
| `factus-backend-contract` | `.agent/skills/factus-backend-contract/SKILL.md` | Calling the backend, invoices, tokens, `reference_code` |
| `excel-parsing-validation` | `.agent/skills/excel-parsing-validation/SKILL.md` | Parsing `.xlsx`, SheetJS, row validation, `factura_id` grouping |
| `react-pwa-conventions` | `.agent/skills/react-pwa-conventions/SKILL.md` | Creating components, screens, state, routing, PWA shell |
| `ts-react-testing` | `.agent/skills/ts-react-testing/SKILL.md` | Writing/running Vitest + Testing Library tests |

---

## 9. Build order (reference)

The overall product is built backend-first; the PWA is **Phase 5** in the plan:

1. Backend connected to Factus sandbox
2. Excel → internal JSON
3. Validator
4. Internal JSON → Factus
5. **PWA (this repo)**
6. Batch processing + results
7. PDF
8. AI auto-mapping (differentiator)

Within this repo, the PWA should still be buildable and testable independently
against a mocked/real backend contract.
