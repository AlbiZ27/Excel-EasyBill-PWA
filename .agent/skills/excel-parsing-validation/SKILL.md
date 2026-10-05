---
name: excel-parsing-validation
description: "Trigger: parsing Excel, SheetJS, xlsx upload, row validation, column mapping, factura_id grouping. Parse and validate the sales Excel in the browser."
license: Apache-2.0
metadata:
  author: excel-easybill-team
  version: "1.0"
---

# Excel Parsing & Validation

## Activation Contract

Apply when reading `.xlsx` files, mapping columns, grouping rows into
invoices, or validating rows before sending to the backend.

## Hard Rules

- Parse `.xlsx` in the browser with SheetJS (`xlsx`). Only `.xlsx` in the MVP
  (no CSV, no Google Sheets).
- Do NOT upload the raw file to the server — send only extracted/validated data.
- Group rows by `factura_id`: rows sharing an id become ONE invoice with
  multiple items. One row is NOT one invoice.
- Keep template field names as the data contract defines them (Spanish):
  `factura_id, fecha, tipo_doc, documento, cliente, email, producto,
  cantidad, precio, iva, forma_pago`.
- Validate every row BEFORE sending; surface fixable per-row errors.

## Decision Gates

| Situation | Action |
|-----------|--------|
| User's columns differ from template | Offer column-mapping UI (user maps → system fields) |
| AI auto-mapping requested | Leave the seam; do NOT build AI (post-MVP) |
| Some rows invalid | Let user fix the Excel OR continue with valid rows only |

## Row Validations

- Valid `fecha`; `documento`, `cliente`, `producto` present
- `cantidad` > 0; `precio` >= 0; valid `iva`; valid `forma_pago`
- Each invoice has at least one product/item

## Execution Steps

1. Read the workbook with SheetJS into row JSON.
2. Apply column mapping (template default or user-provided).
3. Validate each row; collect errors keyed by row number.
4. Group valid rows by `factura_id` into invoices with items arrays.
5. Produce a validation summary (ready vs errored) for preview.

## Output Contract

A grouped, validated invoice set plus a per-row error report, ready for the
preview screen and backend submission.
