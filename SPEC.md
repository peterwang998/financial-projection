# Financial Projection Calculator Spec

## Purpose

This is a single-file, browser-only early retirement and financial independence calculator. It estimates after-tax income, monthly savings, 401(k)/Mega Backdoor Roth routing, years to financial independence, deterministic account balances through age 120, and a Monte Carlo portfolio range.

All major planning outputs are expressed in today's dollars. The calculator is a planning model, not tax, legal, or investment advice.

## Application Shape

- Main file: `calculator.html`
- Runtime: static HTML/CSS/JavaScript in the browser
- Backend: none
- Persistence: save/load a JSON state file
- Primary tabs:
  - `Income & Taxes`
  - `Budget`
  - `Projections`

## Core User Inputs

### Household and Income

- Current age
- Filing mode:
  - Single
  - Head of Household
  - Married Filing Jointly
  - Two Singles (Domestic Partners)
- Gross income by person where relevant
- Pre-tax employee 401(k) deferrals by person where relevant
- Income projection mode:
  - Fixed real income
  - Fixed nominal income, declining in real terms with inflation
  - Annual real growth rate
  - Manual age-by-age income schedule
- Optional early-retirement scenario for one partner
- Optional work break or sabbatical, including decimal-year durations

### Taxes

The calculator models:

- Federal ordinary income tax
- Federal long-term capital gains tax on estimated taxable brokerage income
- California ordinary income tax
- California Behavioral Health Services Tax, formerly Mental Health Services Tax
- Social Security payroll tax
- Medicare payroll tax
- Additional Medicare tax
- Net Investment Income Tax

Editable tax inputs include:

- Federal ordinary brackets
- Federal long-term capital gains brackets
- California brackets
- Federal and California standard or itemized deductions
- Social Security cap/rate
- Medicare and Additional Medicare rates/thresholds
- NIIT rate/threshold
- California Behavioral Health threshold/rate
- Estimated taxable brokerage income yield

Domestic Partners mode uses separate federal filing calculations for Partner A and Partner B, then models California tax as joint California taxation on combined income.

### Budget

Budget lines support:

- `Need`: recurring monthly spending included in current and retirement base spending
- `Want`: recurring monthly spending that can be cut with the cut-wants toggle
- `Recurring`: temporary monthly spending active between start and end ages
- `One-time`: one-time cost assigned to an `atAge`
- `RetirementRecurring`: recurring spending intended for retirement

Each budget item can be marked nominal. Nominal items shrink in real terms over time by inflation. Non-nominal items stay flat in today's dollars.

### Portfolio and Market Assumptions

- Starting taxable, tax-deferred, and Roth balances
- Nominal return assumptions for each account type
- Extra taxable account drag
- Inflation
- Withdrawal rate
- Deferred-account retirement tax haircut
- Portfolio volatility for Monte Carlo
- Employer match rate and match salary cap
- Employee deferral limit
- Overall 401(k) limit

## Calculation Flow

### 1. Income Resolution

For each projected year, `resolveIncome` computes Partner A and Partner B income based on filing mode, income projection mode, work breaks, and early-retirement settings.

### 2. Tax Calculation

`householdTaxes` computes annual taxes from gross income and taxable brokerage balance.

Important current assumptions:

- Employee pre-tax deferrals reduce federal and California income tax wages.
- Employee pre-tax deferrals do not reduce payroll tax wages.
- Taxable brokerage income is estimated as `taxable balance * estimated investment yield`.
- Federal brokerage income is treated as long-term capital gain income.
- California taxes brokerage income as ordinary income.
- NIIT and California Behavioral Health thresholds are nominal and shrink in real terms inside projections.
- Deferred-account withdrawals in retirement are taxed through a flat user-provided haircut, not progressive brackets.

### 3. Savings Routing

Annual net income is converted to monthly income and reduced by active monthly expenses.

If monthly savings are positive:

- Pre-tax employee deferrals go to deferred assets.
- Employer match goes to deferred assets.
- Post-tax savings fill available after-tax 401(k)/Mega Backdoor Roth capacity first.
- Remaining post-tax savings go to taxable brokerage.

If monthly savings are negative:

- The deficit is modeled as a taxable account withdrawal.
- Employer match can still accrue if pre-tax deferrals are configured.

### 4. FI Solver

`solveFI` and `yearsToFI_detailed` simulate up to 60 years.

The FI target is:

```text
annual retirement spending / withdrawal rate
```

The solver also adds estimated remaining temporary costs to the target and iterates healthcare costs because pre-Medicare and post-Medicare retirement healthcare costs depend on the solved retirement age.

### 5. Deterministic Projection

`runFullProjection` generates annual rows from current age through age 120.

Before FI:

- Income, taxes, expenses, savings, and account growth are modeled annually.

After FI:

- Earned income is set to zero.
- Retirement spending, recurring costs, one-time costs, healthcare, brokerage income taxes, and withdrawals are modeled.
- Withdrawal order is taxable, then Roth, then deferred.
- Deferred withdrawals are grossed up by the configured retirement deferred tax rate.

### 6. Monte Carlo

`mcPaths` runs 400 portfolio paths.

Current Monte Carlo assumptions:

- Uses a blended nominal return based on the starting account allocation.
- Converts return to real return by inflation.
- Applies annual cash-flow schedule derived from the deterministic projection.
- Stress mode applies early negative shocks.
- Outputs p10, p50, and p90 end balances and chart bands.

## Main Outputs

- Years to FI
- Retirement age
- Target nest egg in today's dollars
- Monthly needs, wants, and savings
- Savings rate of net income
- Cut-all-wants improvement
- Annual spend at retirement
- Early-retirement impact
- Work-break impact
- Federal, California, payroll, NIIT, and Behavioral Health tax breakdown
- Savings allocation breakdown
- Monte Carlo p10/p50/p90 ending balances
- Deterministic projection table through age 120

## Logic Issues Found and Fixed

### Fixed: Federal deduction treatment for capital gains

Before this change, federal standard or itemized deductions reduced ordinary wage income, but unused deduction did not reduce federal long-term capital gains income. This overtaxed low-ordinary-income or retirement years with large taxable brokerage income.

The calculator now splits federal taxable income into:

- ordinary taxable income after deduction
- taxable preferential capital gains after any unused deduction

Capital gains brackets are then stacked on top of ordinary taxable income using only the taxable capital-gains amount.

Regression check used:

- MFJ, zero wages, $10,000,000 taxable balance, 2% brokerage yield
- Gross brokerage income: $200,000
- Federal standard deduction in code: $31,500
- Taxable federal capital gains: $168,500
- Federal LTCG tax: $10,770

### Fixed: decimal-age one-time events in deterministic projection

Before this change, `runFullProjection` only included one-time events when `age === atAge`. Since projection ages are whole years, a one-time event at age `35.5` was skipped.

The deterministic projection now includes a one-time event when:

```text
age <= atAge < age + 1
```

This matches the FI solver's year-window behavior.

### Fixed: California tax naming

User-facing text now says California Behavioral Health Services Tax and notes that it was formerly the Mental Health Services Tax. The underlying variable names remain `caMH...` to keep the patch small.

## Remaining Logic Risks and Limitations

### RetirementRecurring age bounds are not fully reflected in the FI target

The deterministic projection respects `RetirementRecurring` start and end ages. The FI target currently includes all `RetirementRecurring` monthly amounts as if they apply indefinitely. This can overstate the FI target for temporary retirement-only costs and understate complexity for costs that start later in retirement.

Recommended future fix: model retirement recurring costs as age-bounded cash flows in the FI solver, similar to `Recurring` and `One-time` budget items.

### Tax model is simplified

The calculator does not model:

- AMT
- tax credits
- itemized deduction phaseouts beyond the displayed SALT cap guidance
- ordinary dividends vs qualified dividends
- interest income
- taxable account cost basis
- realized capital gains from selling taxable principal
- Roth conversions
- RMDs
- Social Security benefits and taxation
- Medicare IRMAA
- HSA, IRA, or backdoor Roth IRA flows

### Retirement withdrawal tax model is approximate

Deferred withdrawals use a flat `retireTaxRateDeferred` haircut instead of progressive federal and California tax brackets. This keeps the model simple but can materially misstate retirement taxes.

### Contribution limits do not vary by age

The model uses one employee deferral limit and one overall 401(k) limit per person. It does not model age-50 catch-up contributions, age-60 to age-63 enhanced catch-ups, employer-specific plan limits, or after-tax contribution availability.

### Monte Carlo is portfolio-level

Monte Carlo uses a single blended return and deterministic cash-flow schedule. It does not separately simulate taxable, deferred, and Roth accounts or rebalance allocations over time.

### Domestic partner asset ownership is approximate

Domestic Partners mode splits investment income by wage share when asset ownership is unknown. This may be wrong for households where taxable assets are owned disproportionately by one partner.

## Useful Future Features

1. Add tax-year presets

   Add versioned presets for federal, California, payroll, NIIT, contribution, and deduction values. Include source URLs and a visible "tax year" selector.

2. Split taxable investment income

   Let users divide taxable account yield into qualified dividends, ordinary dividends, interest, and realized long-term gains.

3. Model taxable cost basis

   Track taxable basis so taxable principal withdrawals can estimate realized gains instead of assuming only annual yield is taxable.

4. Add withdrawal optimization

   Compare taxable-first, Roth-first, deferred-first, bracket-filling, Roth-conversion, and ACA/IRMAA-aware withdrawal strategies.

5. Add Social Security and RMDs

   Model Social Security claiming age, benefit taxation, required minimum distributions, and survivor/spouse impacts.

6. Add catch-up and plan-specific contribution rules

   Include age-based catch-ups, employer match vesting, true-up behavior, after-tax 401(k) availability, HSA, IRA, and backdoor Roth IRA flows.

7. Add scenario manager

   Save multiple named scenarios in one JSON file and compare FI age, ending assets, tax paid, and failure-risk outputs side by side.

8. Add validation warnings

   Warn for impossible or suspicious inputs, such as negative ages, end age before start age, withdrawal rates near zero, contribution limits above income, or return assumptions lower than taxable yield plus drag.

9. Extract calculation logic into a testable module

   Move pure calculation functions out of the HTML rendering layer and add automated regression tests for taxes, budget timing, savings routing, FI solving, and projections.

10. Improve retirement spending modeling

   Treat all retirement costs as explicit age-bounded cash flows rather than collapsing them into one permanent spending target.

## External Source Checks

The current code constants are intended to represent 2025 values. Relevant official references checked while preparing this spec:

- IRS federal tax brackets: https://www.irs.gov/filing/federal-income-tax-rates-and-brackets
- IRS 2025 standard deduction: https://www.irs.gov/newsroom/new-and-enhanced-deductions-for-individuals
- IRS retirement plan contribution limits: https://www.irs.gov/newsroom/401k-limit-increases-to-23500-for-2025-ira-limit-remains-7000
- California 2025 Form 540 instructions: https://www.ftb.ca.gov/forms/2025/2025-540-booklet.html
