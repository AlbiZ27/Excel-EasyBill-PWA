---
name: react-pwa-conventions
description: "Trigger: React component, PWA, Vite, folder structure, state management, routing, UI copy. Follow Excel EasyBill PWA structure and component conventions."
license: Apache-2.0
metadata:
  author: excel-easybill-team
  version: "1.0"
---

# React PWA Conventions

## Activation Contract

Apply when creating components, screens, state, routing, or wiring the PWA
shell for Excel EasyBill.

## Hard Rules

- Stack: React + TypeScript + Vite + `vite-plugin-pwa` + SheetJS. Do not
  introduce other frameworks or state libraries without a design decision.
- Code, identifiers, and comments in English. End-user UI copy in Spanish
  (users are Colombian businesses with low technical knowledge).
- Hide technical jargon from the UI (no "OAuth", "API", "JSON", "endpoint").
- Follow the container/presentational split: presentational components are
  pure and prop-driven; containers own data fetching and state.
- The PWA talks only to our backend (see `factus-backend-contract`).

## Decision Gates

| Need | Choice |
|------|--------|
| Routing across screens | `react-router-dom` |
| Shared sequential flow state | Context + `useReducer` or `zustand` (NOT Redux) |
| Schema/data validation | `zod` |
| Backend calls | native `fetch` (add `axios` only if interceptors needed) |

## Screen Flow

Start → Configure Factus → Upload Excel → Detect columns → Validate →
Preview → Confirm → Generate → Results → Download PDF.

## Execution Steps

1. Place screens under `src/screens/`, shared UI under `src/components/`.
2. Keep parsing/validation logic in `src/lib/` (framework-agnostic, testable).
3. Keep backend calls in `src/api/`; never embed Factus logic.
4. Model the flow as explicit steps/state; guard irreversible actions.

## Output Contract

Components matching the container/presentational split, English code with
Spanish UI copy, and the PWA talking only to our backend.
