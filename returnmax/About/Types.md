---
tags:
  - returnmax
  - types
---

# Types

All types in `/types/`. Barrel file `types/index.ts` re-exports everything.

## Aggregate Root

**`TaxReturnState`** (`types/tax-return-types.ts`, ~2882 lines) — the single state object holding all tax return data. Every TTO component reads from/writes to this type through `useTaxReturnStore`.

## Key Domain Types

### Personal Info
- `PersonalInfo` — name, DOB, SSN, occupation, armed forces, uncommon situations (8 toggles), address
- `SpouseInfo` — same fields + death year
- `Dependent` — ~60 fields: name, SSN, DOB, relationship, citizenship, residency, support questions, uncommon situations, qualifying child/relative flags

### Income (15+ form types)
- `W2Form`, `Form1099INT`, `Form1099DIV`, `Form1099B`, `Form1099NEC`, `Form1099MISC`, `Form1099K`, `Form1099R`, `Form1099G`, `SSA1099`
- `K1Form`, `CryptoTransaction`, `SelfEmploymentIncome`, `RentalIncome`, `OtherIncome`
- Each has an `id` (deterministic UUID), `taxYear`, and form-specific fields

### Deductions (25+ categories)
- `Deductions` (top-level container with `method: "standard" | "itemized"`)
- `MortgageInterest`, `SALTDeduction`, `MedicalExpenses`, `StudentLoan1098E`
- `IRAContributions`, `HealthAccountEntry`, `CharitableContribution`
- `CarLoanInterestEntry`, `MovingExpenseEntry`, `CasualtyTheftEvent`
- `EstimatedTaxPayments` (with ~10 sub-objects for federal/state/local/prior-year)
- `SalesTaxDeduction`, `PropertyTaxes`, `EducatorExpenses`, etc.

### Credits (20+ types)
- `Credits` (top-level container)
- `ChildTaxCredit`, `ChildDependentCare`, `EarnedIncomeCredit`, `EducationCredit`
- `EVCredit`, `CleanVehicleCreditEntry`, `EnergyCredit`, `AdoptionCreditEntry`
- `MortgageInterestCreditEntry`, `RetirementSavingsCreditAmount`, `ElderlyDisabledCredit`
- `ForeignTaxCredit`, `PriorYearAmtCredit`, etc.

### Other Tax Situations (17+ form buckets)
- `OtherTaxSituations` (container)
- `AMTInfo`, `IpPinInfo`, `UnderpaymentInfo`, `IdentityTheftInfo`, `ApplyRefundInfo`
- `SelfEmploymentInfo`, `AdditionalTaxesInfo`, `W4EstimatedTaxInfo`, `ChangeOfAddressInfo`
- Each bucket has `hasApplied: boolean` flag

### State Returns
- `StateReturn` — per-state data: adjustments, deductions, credits, status, computed summary
- `StateAdjustments`, `StateDeductionsCredits`

### Filing & Payment
- `FilingState`, `OrderSummary`, `CardPayment`, `EfileState`, `TaxPaymentInfo`
- `FilingSummary` (computed: AGI, taxable income, tax, refund, effective rate)
- `FilingProgress`, `ErrorCheckState`, `ReviewItem`

### Enums & Literal Unions
- `FilingStatus`, `MaritalStatus`, `DependentRelationship`, `DependentRelationType`
- `SectionStatus`, `FilingSection`, `PaymentMethod`, `TaxPaymentMethod`
- `EfileStatus`, `ErrorCheckStage`, `StateReturnStatus`, `StateReturnType`
- `K1EntityType`, `GainLossType`, `EnergyImprovementType`, `CleanVehicleKind`

## UI Types

- `TaxUIState` — current section/page, sidebar, bookmarks, toasts, panels
- `SearchState` — query, results, recent searches
- `ModalState` — title, body, isOpen, callback
- `SettingsState` — boolean toggles

## Store Types

Every store exports a `{ state: XState; actions: XActions }` pair, plus an `XStore` type combining both. All in `types/`.

## Identification Pattern

- Deterministic UUIDs: `generateRecordId(entityType, key1, key2, ...)` — FNV-1a hash seeded from entity type + semantic keys. Used for W-2s, 1099s, deductions, credits, dependents.
- Random UUIDs: `generateId()` (crypto.randomUUID) — used for runtime-generated OTS entries and ad-hoc form records.
