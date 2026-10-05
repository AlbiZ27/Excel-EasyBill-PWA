---
name: factus-backend-contract
description: "Trigger: calling the backend, Factus, invoices, OAuth, tokens, reference_code, generate invoices. Enforce the PWA→Backend→Factus security boundary and request contract."
license: Apache-2.0
metadata:
  author: excel-easybill-team
  version: "1.0"
---

# Factus Backend Contract

## Activation Contract

Apply when writing any code that: connects Factus, sends credentials,
generates/validates invoices, downloads PDFs, or calls our backend.

## Hard Rules

- The PWA MUST NEVER call the Factus API directly.
- Factus OAuth2 credentials and access tokens MUST NEVER be stored in,
  logged by, or passed through the frontend. Tokens expire hourly and are
  the backend's responsibility.
- The only remote the PWA talks to is OUR backend, over HTTPS/JSON.
- If a task asks the PWA to hold a Factus token or hit a Factus endpoint,
  STOP and flag it as a backend concern.
- Every invoice carries a unique internal `reference_code`
  (format `FE-YYYY-NNNNNN`). It is the dedup key — never reuse one.

## Decision Gates

| Situation | Action |
|-----------|--------|
| Need invoice created in Factus | POST data to backend; backend calls Factus |
| Need Factus token refreshed | Do nothing — backend owns it |
| User double-clicks "Generate" | Guard with idempotency on `reference_code` + disable button |
| Need a PDF | Request it from the backend, not from Factus |

## Execution Steps

1. Build the internal invoice JSON (customer, items grouped by `factura_id`).
2. Attach a unique `reference_code` per invoice before sending.
3. Call the backend endpoint over HTTPS; handle per-invoice success/error.
4. On submit, disable the trigger and block re-submission until response.
5. Render results (accepted/rejected) and expose PDF download via backend.

## Output Contract

Code that routes all Factus work through the backend, never exposes secrets
in the frontend, and prevents duplicate submissions via `reference_code`.
