---
name: ts-react-testing
description: "Trigger: tests, Vitest, Testing Library, unit test, test coverage, TDD. Write TypeScript/React tests for Excel EasyBill with Vitest and Testing Library."
license: Apache-2.0
metadata:
  author: excel-easybill-team
  version: "1.0"
---

# TS/React Testing

## Activation Contract

Apply when writing or running tests for logic, components, or flows in the PWA.

## Hard Rules

- Use Vitest + `@testing-library/react`. Run via `vitest --run` (no watch
  mode in automated/agent runs).
- Test behavior and public contracts, not implementation details.
- Pure logic (parsing, validation, grouping) MUST be unit-tested in isolation
  from React — keep it in `src/lib/` so it is trivially testable.
- Query the DOM by role/label/text, not by test-id unless unavoidable.
- Never make real network calls; mock the backend layer in `src/api/`.

## Decision Gates

| Target | Approach |
|--------|----------|
| Excel parsing / validation / grouping | Pure unit tests with fixture rows |
| Component rendering/interaction | Testing Library render + user events |
| Backend-dependent flow | Mock `src/api/` responses |
| Security boundary | Assert no direct Factus call / no token in frontend |

## Execution Steps

1. Co-locate tests as `*.test.ts(x)` next to the unit under test.
2. For validation, cover edge cases: bad `fecha`, `cantidad` <= 0,
   negative `precio`, invalid `iva`/`forma_pago`, multi-row `factura_id`.
3. Mock the backend; assert both success and per-invoice error paths.
4. Run `vitest --run` and fix failures before claiming done.

## Output Contract

Deterministic, network-free tests that cover parsing/validation logic and
key component behavior, runnable with `vitest --run`.
