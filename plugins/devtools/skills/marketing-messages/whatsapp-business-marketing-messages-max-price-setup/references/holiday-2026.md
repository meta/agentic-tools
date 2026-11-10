# Max Price holiday 2026 recommendation guide

Use this guide for holiday/high-value messaging, newly opted-in or dormant cohorts, and pre-holiday test planning. These are **published-rate-relative strategies**, not a live rate card, an optimal-price calculator or permission to spend. Illustrative examples are not current prices or reach forecasts.

## Inputs and execution boundary

Establish the country, account currency, cohort, actual dates/timezone, business objective and approved financial ceiling. For rate-relative advice, obtain the current published rate with source/as-of information. For reach-based decisions, use supported estimates for the same cohort, country and period; business-supplied aggregate estimates must be labeled as supplied, not MCP-fetched.

The account/template tools in this skill do not fetch rate cards or reach estimates and do not accept a rate-factor recommendation for monetary normalization. Never invent an endpoint, estimate or price. Missing inputs allow an explanation of the strategy or an explicit-value V2 setup, not a claim that a live-data-backed bid has been calculated. Stop the estimate-based recommendation when its evidence is unavailable, stale or mismatched.

Even when the numbers coincide because the base bid equals the published rate, that equivalence must be established in the same currency/units; never assume it from the guide. A published-rate factor is **not automatically a country override multiplier**. Server-owned logic must resolve the target price as published rate × recommended factor, then express any country override relative to the configured base bid in the same currency and units. Do not copy a rate factor directly into `bid_country_multiplier_overrides`, pass it as `bid_amount_major`, or replace missing server normalization with model arithmetic. Until that capability exists, ask the business for explicit monetary settings in account-currency major units per 1,000 deliveries and preview them through the existing V2 tool.

Max Price is the maximum willingness to pay per delivered message; charges may be lower. It is not a total spending budget. Every applied setting remains bound to exact-preview approval. Country scope is not recipient selection. No message sending, automatic repricing/restoration, or experiment execution is performed by this skill.

## Holiday / in-demand, high-value audiences

Use the business's stated objective to select a strategy; clarify only if ambiguous. Neither strategy establishes an unlimited ceiling or guarantees delivery/ROI.

### Ladder up

1. Inspect estimated deliveries at the published rate (1.0× baseline).
2. Consider **1.0–1.5×** of the published rate within the approved ceiling. If the estimate meets the business's target, **hold**.
3. Otherwise consider **1.5–2.0×** and re-check estimates for that same audience and period.
4. Consider higher values only when estimates remain unsatisfactory, the business explicitly accepts the financial ceiling, and supporting estimates are available.
5. **Stop at the plateau:** when a higher tested price yields no improvement, do not keep raising it. Prefer the lower tested price with the same satisfactory estimate. Also stop when the target is met, the approved cap is reached, or the evidence is unavailable.

This is a bounded recommendation sequence over supported estimates, not a repeated-send or repeated-write loop. A holiday does not authorize price changes or choose an exact point within a range.

### Leap

For a business prioritizing a single peak moment with high-value audiences, explain using its explicitly approved comfortable ceiling in one pass. Do not automatically pick 2.0×, 2.5×, a platform maximum, or a figure from an illustrative example. If the user supplies an explicit supported monetary cap, it can enter the normal V2 preview flow; without a current published rate, do not label that cap as a known rate-card multiple.

## Expansion / re-engagement cohorts

Start with supported reach estimates above and below the published rate for the actual cohort. The business or its existing integration owns opt-in status, audience membership and send routing; do not collect individual recipient data here.

### Newly opted-in customers

These customers may have high intent. Recommend considering **above-published-rate** pricing to prioritize delivery while intent is high, using the audience-specific estimate and approved ceiling. Do not assume that intent or conversion is guaranteed, or apply a universal 1.5× factor merely because an example used it.

### Dormant or lightly engaged customers

Recommend exploring **below-published-rate** pricing for economical re-engagement. Compare supported lower-price estimates or supplied test observations. Once a tested lower price reaches zero deliveries, the last tested positive-delivery point is the **lowest observed useful tested price**, not a guaranteed global minimum. Choose a price that still fits the business's reach objective and approved ceiling; do not invent intermediate results, treat 50%/60% as a universal floor, or claim an untested lower value would work.

New-country expansion still needs local evidence; same-country cohort expansion still needs verified audience/template routing. Country multipliers alone cannot isolate newly opted-in versus dormant customers using the same template in the same country. Additional WhatsApp sends are not proof of incremental reach, ROI or business value.

## Pre-holiday A/B test plan

Offer a reviewed plan, not execution. The recommended design uses **40,000 total send requests**, not guaranteed deliveries: randomly split **40,000 opted-in recipients into four equal arms of 10,000**, ideally within one day. Reuse **one identical template**, schedule and configuration; only the Max Price treatment differs. Compare per-arm delivery rates using per-arm send requests as the denominator, and supported cost/outcome signals when available.

| Arm | Treatment |
|------|-----------|
| A | Business as usual / fixed-price control, without the Max Price treatment |
| B | 1.0× of the current published rate |
| C | Above-rate treatment from the country matrix below |
| D | Below-rate treatment from the country matrix below |

| Country | B: rate card | C: above | D: below |
|---------|--------------|----------|----------|
| IN | 1.0 | 1.2 | 0.9 |
| SA | 1.0 | 1.2 | 0.7 |
| Other | 1.0 | 1.5 | 0.7 |

The IN/SA exception is for the **above-rate test arm**; the below-rate exception applies to **IN only**. Here **SA means Saudi Arabia**, not South Africa: South Africa (ZA) follows the Other row. This test matrix is not the ladder-up range, the leap ceiling or a universal cohort recommendation. Apply factors to the current published rate for the relevant country/currency, not an illustrative price or an existing base bid.

### Capability gate before producing executable instructions

The current MCP tools configure Max Price on a template. Do not assume the existing Direct-send integration can apply four different per-arm prices while reusing one template. Verify an authorized per-arm pricing mechanism first. If it is unavailable, flag the implementation gap and retain a test plan only; do not silently substitute duplicate templates, country overrides for same-country arms, repeated global template edits, or overlapping sends.

Do not reset a production template to manufacture the control arm, upload recipients, call send tools, or perform the experiment. Its isolation, cohort assignment and execution belong to the business's supported integration under separate authorization.

## Recommendation report

Show the scenario/strategy, country and cohort scope, recommended factor/range or explicit user cap, rate/estimate provenance and freshness, business ceiling, rationale and stopping decision. Separate guide policy from observed evidence and illustrative examples. State which required data or execution capability is missing. Only present normalized monetary write arguments once they are supported and validated, then use the skill's unchanged preview/approval/readback flow.

Do not promise the guide's historical cost savings to this business or equate delivery with incrementality. Recommendations remain directional; delivery and business outcomes are not guaranteed.
