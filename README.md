# SIA "Limited Seats" Banner — A/B Test Simulation

A production-grade A/B testing notebook simulating a conversion rate experiment for Singapore Airlines (SIA). The experiment tests whether displaying a **"Limited seats left at this price"** urgency banner on the flight search results page increases booking conversion.

Built from scratch with deliberate, justified design decisions at every step — not just statistical correctness, but commercial defensibility and production realism.

---

## Experiment Overview

| Parameter | Value |
|---|---|
| Baseline conversion rate | 5.0% |
| Minimum Detectable Effect | 6.5% (~1.5pp lift) |
| Significance level (α) | 0.05 |
| Power target | 90% (upgraded from 80%) |
| Minimum experiment duration | 14 days |
| Estimated daily sessions | 10,000 |

---

## Notebook Structure

| Section | What It Does |
|---|---|
| **1. Imports & Parameters** | Sets all experiment constants in one place. Uses `numpy.random.default_rng` (PCG64, modern standard) instead of legacy `np.random.seed`. |
| **2. Power Analysis** | Calculates required sample size using Cohen's *h* (arcsine transformation), which correctly handles proportions near 0 and 1 — unlike raw difference. |
| **3. Power Curve** | Visualises the risk of stopping the experiment early. Shows the underpowered region to stakeholders. |
| **4. Power Upgrade (80% → 90%)** | Justifies upgrading the power target because SIA's traffic volume makes the additional sample size commercially cheap. Enforces a 14-day minimum to wash out novelty effects. |
| **5. Simulating SIA Visitor Data** | Generates realistic SIA-style data with full UUIDs (`SIA-<uuid>`), deterministic group assignment via `hashlib.md5`, `Gamma(5, 60)` session durations (~5 min mean), and 60/40 mobile/desktop split. |
| **6. Randomisation Checks** | Five pre-analysis checks: group size balance (SRM), session duration balance, device type balance, day-of-week balance, and user uniqueness. |
| **7. Conversion Analysis** | Two-proportion Z-test, Wilson CI for individual rates, Wald CI for the difference. Decision gate requires **both** statistical significance (p < 0.05) and practical significance (≥5% relative lift). |
| **8. Visualisation** | Three-panel chart: conversion rates with Wilson CI, effect size with Wald CI (dot-and-whisker format), conversion rate by device type. |
| **9. Summary Report** | Self-contained stakeholder summary with experiment overview, results, interpretation, decision, and four production caveats. |

---

## Key Design Decisions

### Why Cohen's *h* instead of raw difference?
A jump from 5% → 6.5% is statistically harder to detect than 50% → 51.5%, even though both are 1.5pp. The arcsine transformation in Cohen's *h* corrects for this asymmetry. Using raw difference gives a slightly wrong sample size.

### Why `hashlib.md5` for group assignment?
Python's built-in `hash()` is randomised across kernel restarts (Python 3.3+). If a user returns mid-experiment, pure random assignment would swap their group, contaminating results. `hashlib.md5` is deterministic across runs, machines, and Python versions — exactly how production systems like Optimizely and Statsig work under the hood.

### Why full UUIDs (`SIA-<uuid>`) instead of sequential integers?
Real user IDs are alphanumeric strings, not clean integers. Sequential integers are structurally biased — a naive `user_id % 2` split would perfectly alternate groups, which is not how real traffic arrives. Hashing a UUID breaks any underlying pattern before splitting.

### Why `numpy.random.default_rng` instead of `np.random.seed`?
`default_rng` creates an isolated Generator object using PCG64 — statistically superior to the older Mersenne Twister and reproducible via explicit seed. The legacy global seed can be silently corrupted by third-party imports.

### Why 14 days minimum?
A "Limited seats" banner will see inflated conversion in Week 1 because it is novel and surprising, not because it genuinely influences booking decisions. Two full weeks ensures the novelty effect washes out and the measured lift reflects true steady-state behaviour.

### Why upgrade from 80% to 90% power?
The power curve confirms 80% is the industry standard and sufficient. However, at 10,000 daily sessions, achieving 90% power costs only a few additional days. At SIA's scale, missing a genuine conversion lift is commercially more expensive than running slightly longer — so we take the upgrade.

### Why both statistical and practical significance gates?
A p-value of 0.049 on a 0.001pp lift is statistically significant but commercially worthless. The decision gate requires the observed relative lift to also meet the 5% minimum threshold set by the revenue team.

---

## Five Randomisation Checks (Cell 6)

Before touching any conversion data, the notebook verifies the simulation is fair:

1. **Group Size Balance** — Chi-square SRM test. A mismatch here invalidates everything downstream.
2. **Session Duration Balance** — T-test. If treatment users are more engaged *before* seeing the banner, any lift is contaminated.
3. **Device Type Balance** — Chi-square. Mobile and desktop behave differently in booking funnels.
4. **Day-of-Week Balance** — Chi-square. Weekend traffic behaves differently from weekday traffic. If one group accidentally receives more Saturday sessions, session duration is biased.
5. **User Uniqueness** — Confirms every row is a distinct user. A tracking bug causing duplicate user IDs inflates sample size and breaks t-test assumptions.

---

## Caveats

1. **Novelty effect** — Monitor conversion weekly post-launch. If lift decays after 4 weeks, the effect may be novelty-driven, not genuine urgency response.
2. **Segmentation** — Validate the effect holds across route types (short-haul vs long-haul) and booking windows (same-day vs advance).
3. **No peeking** — This result is valid only because it was read once, at the predetermined sample size. Do not rerun the test and cherry-pick a favourable window.
4. **Inventory accuracy** — The banner must only fire when seats are genuinely limited. False scarcity is a brand risk for SIA. Losing customer trust is more damaging than any conversion lift.

---

## Requirements

```
numpy
pandas
matplotlib
seaborn
scipy
statsmodels
```

Install with:

```bash
pip install numpy pandas matplotlib seaborn scipy statsmodels
```

---

## Running the Notebook

1. Clone this repository
2. Install dependencies (see above)
3. Open `sia_conversion_ab_test.ipynb` in Jupyter or Google Colab
4. Run all cells top to bottom (`Kernel > Restart & Run All`)

> **Note:** Cells must be run in order. Each cell depends on variables set in earlier cells.

---

## Author Notes

This notebook was built cell by cell as a learning exercise, with every technical decision explicitly justified. It is designed to be readable by both data scientists and non-technical stakeholders.

The simulation uses realistic but synthetic data. In a live SIA deployment, `P_CONTROL`, `DAILY_SESSIONS`, and `MIN_RELATIVE_LIFT` would be replaced with values from the analytics dashboard and signed off by the revenue team before the experiment begins.
