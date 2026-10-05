# Skill Registry — Excel EasyBill (PWA)

Index of shared agent skills. Match by trigger, then load the exact `SKILL.md`
path before doing the work. Skills are LLM-first instruction contracts, not
runtime code or dependencies.

| Skill | Scope | Triggers | Path |
|-------|-------|----------|------|
| `factus-backend-contract` | project | calling the backend, Factus, invoices, OAuth, tokens, reference_code, generate invoices | `.agent/skills/factus-backend-contract/SKILL.md` |
| `excel-parsing-validation` | project | parsing Excel, SheetJS, xlsx upload, row validation, column mapping, factura_id grouping | `.agent/skills/excel-parsing-validation/SKILL.md` |
| `react-pwa-conventions` | project | React component, PWA, Vite, folder structure, state management, routing, UI copy | `.agent/skills/react-pwa-conventions/SKILL.md` |
| `ts-react-testing` | project | tests, Vitest, Testing Library, unit test, test coverage, TDD | `.agent/skills/ts-react-testing/SKILL.md` |
