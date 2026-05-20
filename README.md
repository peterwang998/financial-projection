# Financial Projection Calculator

A browser-based early retirement and financial independence calculator. It models household income, federal and California taxes, budget-driven savings, 401(k)/Mega Backdoor Roth routing, deterministic projections, and Monte Carlo portfolio ranges.

Hosted calculator: https://peterwang998.github.io/financial-projection/

## Quick Start

1. Open the hosted calculator or open `calculator.html` directly in a browser.
2. Start on `Income & Taxes` and enter your household, income, tax, portfolio, and market assumptions.
3. Go to `Budget` and add monthly spending line items.
4. Set retirement spending and healthcare assumptions.
5. Open `Projections` to review years to FI, retirement age, portfolio ranges, and the deterministic projection table.
6. Use `Save as File` to export your scenario as JSON. Use `Load from File` later to restore it.

The calculator runs entirely in the browser. It does not send your financial data to a server.

## What It Calculates

- After-tax annual income
- Federal ordinary income tax
- Federal long-term capital gains tax on estimated taxable brokerage income
- California ordinary income tax
- California Behavioral Health Services Tax
- Social Security, Medicare, and Additional Medicare payroll taxes
- Net Investment Income Tax
- Monthly savings after budgeted spending
- Pre-tax 401(k), employer match, Mega Backdoor Roth, and taxable savings routing
- Years to financial independence
- Estimated retirement age
- Target nest egg in today's dollars
- Deterministic account balances through age 120
- Monte Carlo p10, p50, and p90 portfolio outcomes

## Core Concepts

### Today's Dollars

Most outputs are shown in today's dollars. A non-nominal expense or income item stays flat in real purchasing power. A nominal item keeps the same fixed dollar amount, so its real cost falls over time as inflation compounds.

### Financial Independence Target

The FI target is based on:

```text
annual retirement spending / withdrawal rate
```

The solver also adds remaining temporary costs and iterates healthcare costs because pre-Medicare and post-Medicare healthcare costs depend on the projected retirement age.

### Savings Routing

Post-tax savings are automatically routed in this order:

1. Available after-tax 401(k) space for Mega Backdoor Roth
2. Taxable brokerage

Pre-tax employee deferrals and employer match are routed to deferred assets. The calculator enforces the configured employee deferral limit and overall 401(k) limit per person.

## Using the Income & Taxes Tab

### Core Assumptions

Set:

- Current age
- Inflation
- Real withdrawal rate
- Monte Carlo volatility
- Stress test toggle

`Stress test` front-loads early negative market shocks in the Monte Carlo simulation.

### Save, Load, and Reset

- `Save as File` exports the current calculator state as JSON.
- `Load from File` imports a saved JSON scenario.
- `Reset` restores the built-in default assumptions.

When loading an older file, the app may ask whether to keep saved tax brackets and limits or replace them with the current defaults.

### Filing Mode

Supported filing modes:

- `Single`
- `Head of Household`
- `Married Filing Jointly`
- `Two Singles (Domestic Partners)`

For `Married Filing Jointly`, enter Partner A and Partner B income and pre-tax contributions separately. Combined gross income is derived automatically.

For `Two Singles (Domestic Partners)`, the calculator models separate federal returns for Partner A and Partner B, then models California tax on joint combined income.

### Income Inputs

Enter gross annual income and annual pre-tax employee contributions. Pre-tax contributions reduce federal and California income-tax wages but do not reduce payroll-tax wages.

Income projection modes:

- `Fixed`: income stays flat in today's dollars.
- `Nominal`: income stays fixed in nominal dollars and loses purchasing power over time.
- `Annual real growth rate`: income grows by a real growth percentage above inflation.
- `Manual schedule`: income changes at specific ages using the schedule table.

### Early Retirement and Work Breaks

For multi-person filing modes, enable `Early Retirement Scenario` to stop one partner's income at a selected age.

Enable `Work Break / Sabbatical` to model a temporary pause in earned income. Decimal durations are supported, such as `0.5` for a half-year break.

### Taxes and Brackets

The calculator includes editable defaults for:

- Federal ordinary brackets
- Federal long-term capital gains brackets
- California brackets
- Federal deductions
- California deductions
- Social Security rate and wage cap
- Medicare and Additional Medicare rates
- Employee deferral and overall 401(k) limits
- NIIT rate and threshold
- Estimated taxable brokerage income yield
- California Behavioral Health Services Tax threshold and rate

Taxable brokerage income is estimated as:

```text
taxable brokerage balance * taxable brokerage income yield
```

That amount is treated as long-term capital gains federally and ordinary income for California.

### Tax Breakdown

Use `Show tax breakdown` to inspect:

- Gross income
- Pre-tax contributions
- Federal ordinary taxable income
- Federal taxable capital gains
- Federal ordinary and capital gains tax
- California taxable income
- California bracket tax
- California Behavioral Health Services Tax
- Payroll taxes
- NIIT

## Using the Portfolio & 401(k) Routing Section

Enter starting balances for:

- Taxable
- Deferred
- Roth

Set return and savings-routing assumptions:

- Taxable nominal return
- Deferred nominal return
- Roth nominal return
- Residual taxable drag
- Employer match rate
- Match up to percent of salary

Use residual taxable drag only for extra taxable-account drag not already captured by explicit taxable brokerage income taxes. In many cases this can be set to `0%`.

## Using the Budget Tab

Add one row per spending item. Each row has:

- Line item name
- Monthly amount
- Type
- Nominal checkbox
- Start age
- End age
- At age
- FI impact estimate

### Budget Item Types

`Need`

Recurring monthly spending included in current spending and retirement base spending.

`Want`

Recurring monthly spending included unless `Cut all wants` is enabled.

`Recurring`

Temporary monthly spending between start and end ages. Use this for costs such as childcare, rent overlap, temporary family support, or a multi-year loan.

`One-time`

One-time spending at a specific age. Decimal ages are supported.

`RetirementRecurring`

Recurring spending intended for retirement. This is useful for costs that should be added to the retirement spending target.

### Nominal Checkbox

Check `Nominal` for fixed-dollar costs that do not rise with inflation, such as a fixed mortgage payment. In real terms, these costs shrink over time.

Leave it unchecked for costs that should remain constant in today's purchasing power.

### Cut All Wants

`Cut all wants` removes `Want` spending from the plan and routes the freed cash flow into savings, subject to the account-routing caps.

### Impact Column

The impact column estimates how many extra years each expense adds to the FI timeline. It is calculated by removing that line item and resolving the plan.

## Retirement Spending and Healthcare

In the `Budget` tab, set:

- Retirement annual spending in today's dollars
- Medicare age
- Pre-Medicare monthly healthcare cost
- Post-Medicare monthly healthcare cost

If retirement annual spending is blank, the calculator defaults to permanent monthly `Need` plus `Want` spending annualized.

Healthcare is added to the retirement spending target. The solver iterates because whether you retire before or after Medicare age changes which healthcare cost applies.

## Using the Projections Tab

### Summary Outputs

The top summary shows:

- Years to FI
- Retirement age
- Target nest egg
- Monthly needs
- Monthly wants
- Monthly savings
- Savings rate of net income
- Cut-all-wants improvement
- Annual spend at retirement

If early retirement or work break scenarios are enabled, this tab also shows their impact on the FIRE timeline.

### Monte Carlo Chart

The portfolio range chart shows 400 random-return runs in today's dollars:

- p10 ending balance
- p50 ending balance
- p90 ending balance
- shaded p10-p90 range
- median path
- deterministic baseline path

Use `Project until age` to change the chart horizon.

### Deterministic Baseline Projection

The deterministic projection table runs from current age through age 120 and shows:

- Age
- Total assets
- Investment income
- Earned income by partner
- Expenditure
- Savings
- Net change

The deterministic model uses the configured expected return assumptions rather than random returns.

## Local Development

This is a static site. No build step is required.

To run locally, open:

```text
calculator.html
```

or use any static file server from the repo root.

The GitHub Pages entry point is:

```text
index.html
```

It redirects to `calculator.html`.

## Saving and Sharing Scenarios

Use `Save as File` to download a JSON state file. This file contains your calculator inputs and can be loaded later with `Load from File`.

Do not commit personal scenario JSON files unless you intentionally want to publish that data.

## Modeling Limits

The calculator is intentionally simplified. It does not currently model:

- AMT
- tax credits
- ordinary dividends vs qualified dividends
- interest income
- taxable account cost basis
- capital gains caused by selling taxable principal
- Roth conversions
- Social Security benefits and taxation
- required minimum distributions
- Medicare IRMAA
- age-based catch-up contributions
- HSA, IRA, or backdoor Roth IRA flows

Deferred-account withdrawals in retirement use a flat tax haircut rather than progressive tax brackets.

Domestic Partners mode splits investment income by wage share when asset ownership is unknown.

See `SPEC.md` for the detailed calculation spec, known limitations, and future feature ideas.
