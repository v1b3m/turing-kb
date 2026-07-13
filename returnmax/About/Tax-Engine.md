---
tags:
  - returnmax
  - tax-engine
  - lib
---

# Tax Engine & Business Logic

All in `/lib/`. ~95 files covering tax calculations, form normalization, wizard flows, and utilities.

## Core Calculation

### `tax-engine.ts`
Central `computeFilingSummary()` function. Computes:
- Total income (sum all income sources)
- Adjustments (IRA, HSA, student loan interest, self-employment, alimony, educator)
- AGI = income − adjustments
- Deduction (standard or itemized, whichever is larger)
- Taxable income = AGI − deduction
- Tax (2025 bracket rates)
- Credits (child tax, dependent care, EIC, education, retirement savings, energy, EV, etc.)
- Payments (withholding, estimated tax)
- Refund or balance due
- Effective tax rate, marginal bracket

### `tax-constants.ts`
2025 tax year constants: SALT cap ($10,000), standard deduction amounts per filing status, child tax credit amount ($2,000), EIC parameters.

### `compute-state-return-summary.ts`
Per-state return computation. Sums adjustments (additions − subtractions), deductions, credits from detail fields. Computes `stateRefundOrOwed`.

## Income Processing

| File | Purpose |
|------|---------|
| `form-1099-sa.ts` | Normalize 1099-SA forms |
| `holding-period.ts` | Capital asset holding period (short/long-term) |
| `withholding-not-on-w2.ts` | Backup withholding flow |

## Deduction Logic

| File | Purpose |
|------|---------|
| `deductions-normalize.ts` | `normalizeDeductions()` — restructures deduction object on store rehydrate |
| `deductions-onboarding.ts` | "Find tax breaks" onboarding questions |
| `medical-expenses.ts` | Medical expense deduction (7.5% AGI threshold) |
| `student-loans.ts` | Student loan interest ($2,500 cap, income phaseout) |
| `ira-contributions.ts` | IRA contribution limits and deductibility |
| `charitable-wizard.ts` | Charitable contribution wizard flow |
| `mortgage-wizard.ts` | Mortgage interest (Form 1098) wizard |
| `sales-tax-deduction.ts` | Sales tax (actual vs IRS table) |
| `property-taxes.ts` | Property tax deduction |
| `educator-expenses.ts` | Educator expense ($300/$600 limit) |

## Credit Logic

| File | Purpose |
|------|---------|
| `child-tax-credit.ts` | CTC eligibility and qualification |
| `dependent-care-credits.ts` | Child/dependent care credit |
| `earned-income-credit.ts` | EIC computation with phase-in/phase-out |
| `education-credits-wizard.ts` | AOTC/LLC education credits |
| `adoption-credits.ts` | Adoption credit + normalization |
| `clean-vehicle-credits.ts` | New/used/commercial clean vehicle credit |
| `energy-credits.ts` | Energy efficient home improvement credit |
| `retirement-savings-credit.ts` | Saver's Credit |
| `foreign-tax-credit.ts` | Foreign tax credit (Form 1116) |
| `elderly-disabled-credit.ts` | Credit for elderly/disabled |

## Flow Wizards (multi-step form orchestrators)

Each wizard file exports step definitions, answer snapshots, and transition logic:

`mortgage-wizard.ts`, `medical-wizard.ts`, `ira-wizard.ts`, `hsa-wizard.ts`, `charitable-wizard.ts`, `energy-wizard.ts`, `adoption-wizard.ts`, `child-care-wizard.ts`, `education-credits-wizard.ts`, `retirement-savings-wizard.ts`, `clean-vehicle-credit-wizard.ts`, `ev-charging-wizard.ts`, `car-loan-interest-wizard.ts`, `esa-529-distributions-wizard.ts`, `coverdell-esa-excess-wizard.ts`, `dc-first-time-homebuyer-flow.ts`

## Estimated Tax Payments

Complex multi-flow system: `estimated-tax-payments.ts`, `federal-estimated-taxes-2025.ts`, `state-estimated-taxes-2025.ts`, `local-estimated-taxes-2025.ts`, `prior-year-state-estimated-2024.ts`, `prior-year-local-estimated-2024.ts`, `tax-extension-payments.ts`, `prior-year-state-local-payments.ts`

## Utilities

| File | Purpose |
|------|---------|
| `utils.ts` | `cn()` (Tailwind class merge), `generateId()`, `generateRecordId()`, `formatPhone()` |
| `safe-storage.ts` | SSR-safe localStorage wrapper with custom event dispatch |
| `input-formats.ts` | SSN, EIN, phone, ZIP, currency input masking |
| `format-money.ts` | Currency display formatting |
| `date-utils.ts` | Date formatting |
| `us-states.ts` | US states/territories list |
| `search-topics.ts` | Tax topic search index and filtering |
| `unified-routes.ts` | Route path constants |
| `reset-channel.ts` | BroadcastChannel-based reset coordination |

## Validation

| File | Purpose |
|------|---------|
| `personal-info-validation.ts` | `hasMissingPersonalInfo()` |
| `payment-validation.ts` | Card/banking validation |

## Contracts

`lib/contracts/state-schema.json` — JSON Schema for RL Gym state validation. `state-schema.ts` re-exports it.
