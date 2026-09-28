# ForS Works — Year 1 Budget Model

**Purpose:** planning baseline for a one-person, handcrafted 3D-model business using AI workers for design support, marketing, customer support, and operations.

**Currency:** USD, before sales tax and income tax. Amounts are cash-planning estimates, not accounting or tax advice.

## Executive recommendation

Use the **base case of $24,000 for Year 1 operating spend** (about **$2,000/month**). Keep a separate **$3,600 contingency reserve** (15%) so the business does not have to pause production after an equipment failure, rework batch, or weak sales month.

The original business-plan percentages are retained as the allocation target:

| Category | Target | Year 1 base | Monthly average |
| --- | ---: | ---: | ---: |
| Production | 48% | $11,520 | $960 |
| Marketing | 20% | $4,800 | $400 |
| Operations & logistics | 15% | $3,600 | $300 |
| R&D / AI tooling | 10% | $2,400 | $200 |
| Miscellaneous | 7% | $1,680 | $140 |
| **Total operating budget** | **100%** | **$24,000** | **$2,000** |

## Assumptions

- The owner is the primary maker; owner salary, rent, and personal living costs are excluded. They should be added before treating this as a full company budget.
- Initial production is a small batch of **20 sellable units/month**, scaling toward 40 units/month if demand supports it.
- Base average order value is **$150** before shipping and sales tax. At a 55% gross margin after direct materials and packaging (45% direct-cost rate), each order contributes about **$82.50** before marketing and overhead.
- Payment and marketplace fees are modeled at **8% of revenue** and are included in operations/logistics. Shipping charged to customers is assumed to cover shipping cost on average; subsidized shipping is a downside risk.
- Production includes materials, paint/consumables, packaging, replacement tools, and small-batch equipment amortization. It does not include the owner's labor.
- AI tooling is a capped budget for model/API subscriptions, image generation, automation, storage, and worker evaluation. No worker may run an uncapped retry loop.
- Marketing starts at $250/month and increases only after a channel produces attributable orders; the $4,800 annual target is a ceiling, not a commitment to spend it all.

## Monthly run-rate detail — base case

| Category | Included costs | Typical monthly amount |
| --- | --- | ---: |
| Production | Resin/filament/plaster, paints, tools, packaging, failed-piece allowance | $960 |
| Marketing | Paid tests $250, creator samples $75, content/software $50, email/community $25 | $400 |
| Operations & logistics | Marketplace/payment fees $120, shipping supplies/postage subsidy $100, domain/site $30, bookkeeping/legal reserve $50 | $300 |
| R&D / AI tooling | LLM/API worker calls $80, image/design tools $50, automation/hosting $40, evaluation/experiments $30 | $200 |
| Miscellaneous | Insurance, refunds, bank charges, small admin purchases | $140 |
| **Total** |  | **$2,000** |

Production and marketing are intentionally variable. In a month with no confirmed orders, production should fall to roughly **$450** for maintenance and samples, while paid marketing should pause at the experiment minimum of **$100**.

## Scenario range

| Scenario | Operating budget | Reserve (15%) | Cash needed for Year 1 | What it means |
| --- | ---: | ---: | ---: | --- |
| Lean validation | $15,000 | $2,250 | **$17,250** | 10–15 units/month, organic-first marketing, existing equipment |
| Base launch | $24,000 | $3,600 | **$27,600** | 20–40 units/month, measured paid tests, modest tooling upgrades |
| Growth | $36,000 | $5,400 | **$41,400** | 40–70 units/month, more inventory, creator partnerships, higher support load |

The growth case should not be funded upfront. Release it in quarterly tranches after the business meets the gates below.

## Revenue and break-even checks

At the base assumptions, contribution per order is:

`$150 average order value × (1 − 45% direct-cost rate − 8% fees) = $70.50`

This contribution is **after direct production cost and transaction fees**, but before the fixed/semifixed budget. Covering the $2,000 monthly operating budget therefore requires approximately **29 orders/month** (`$2,000 ÷ $70.50`), or **$4,350/month revenue**. If owner labor is valued at $25/hour and takes 4 hours/order, add $100/order to the economic cost; the business then needs either a higher price, fewer labor hours, or a custom-order premium.

## AI worker cost guardrails

- Set a hard monthly ceiling of **$200** in the base case and alert at 70% usage.
- Reserve approximate monthly capacity of: research/content $50, customer support $40, design iteration $60, automation/evaluation $30, and contingency $20.
- Route routine drafts to the lowest-cost model; use a stronger model only for final review or complex design work.
- Log runs by task, token/API cost, and outcome. Stop or retry at most twice on timeout/error.
- Do not send customer personal data, payment data, credentials, or unpublished IP to external model providers.

## Release gates and review cadence

- **Monthly:** compare actual spend and orders to budget; pause any channel with no attributable signal after two test cycles.
- **Quarterly growth gate:** release the next growth tranche only if gross margin is at least 55%, on-time fulfillment is at least 95%, and the 3-month cash runway remains above 90 days.
- **Reserve use:** approve only for equipment failure, material price shock, refunds/rework, or a validated demand opportunity. Replenish the reserve before increasing discretionary marketing.

## Exclusions to add before financing or hiring

Owner compensation, payroll taxes, commercial insurance quotes, storage/workshop rent, depreciation, sales-tax remittance, income tax, returns beyond the 3% defect allowance, international duties, and debt financing are not included. These can materially change the required budget and break-even price.
