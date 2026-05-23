# SIA "Limited Seats" Banner — A/B Test Simulation 📊

**Production-grade A/B testing notebook for Singapore Airlines conversion optimisation**

[![Python](https://img.shields.io/badge/Python-3.11-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)](https://jupyter.org/)
[![Open in Colab](https://img.shields.io/badge/Open_in-Google_Colab-F9AB00?style=for-the-badge&logo=googlecolab&logoColor=white)](https://colab.research.google.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)](LICENSE)

> **Built with:** NumPy · Pandas · SciPy · Statsmodels · Matplotlib · Seaborn

---

## 🎯 What It Tests

Does displaying a **"Limited seats left at this price"** urgency banner on the Singapore Airlines flight search results page increase booking conversion?

This notebook simulates the full end-to-end A/B test — from power analysis to a stakeholder-ready summary report — with every design decision explicitly justified.

**This isn't a textbook stats exercise.** Every choice reflects how real experimentation teams at companies like Optimizely, Statsig, and Airbnb actually run experiments.

---

## 📐 Experiment Parameters

| Parameter | Value |
|---|---|
| Baseline conversion rate | 5.0% |
| Minimum Detectable Effect (MDE) | 6.5% (~1.5pp relative lift) |
| Significance level (α) | 0.05 |
| Power target | 90% (upgraded from 80%) |
| Minimum experiment duration | 14 days |
| Estimated daily sessions | 10,000 |

---

## 🏗️ Notebook Structure

| Section | What It Does |
|---|---|
| **1. Imports & Parameters** | All experiment constants in one place. Uses `numpy.random.default_rng` (PCG64) — not legacy `np.random.seed`. |
| **2. Power Analysis** | Sample size via Cohen's *h* (arcsine transformation) — correct for proportions near 0 and 1, unlike raw difference. |
| **3. Power Curve** | Visualises underpowered risk. Shows stakeholders exactly what happens if the experiment is stopped early. |
| **4. Power Upgrade (80% → 90%)** | Justifies the upgrade commercially: at 10,000 daily sessions, 90% power costs only a few extra days. Enforces 14-day minimum for novelty effect washout. |
| **5. Simulating SIA Visitor Data** | Realistic synthetic data: full UUIDs (`SIA-<uuid>`), deterministic group assignment via `hashlib.md5`, `Gamma(5, 60)` session durations (~5 min mean), 60/40 mobile/desktop split. |
| **6. Randomisation Checks** | Five pre-analysis integrity checks before touching conversion data. |
| **7. Conversion Analysis** | Two-proportion Z-test, Wilson CI for individual rates, Wald CI for the difference. Dual gate: statistical AND practical significance. |
| **8. Visualisation** | Three-panel chart — conversion rates with Wilson CI, effect size with Wald CI (dot-and-whisker), conversion by device type. |
| **9. Summary Report** | Self-contained stakeholder summary: overview, results, interpretation, decision, and four production caveats. |

---

## 🔑 Key Design Decisions

### Why Cohen's *h* instead of raw difference?
A jump from 5% → 6.5% is statistically harder to detect than 50% → 51.5%, even though both are 1.5pp. The arcsine transformation in Cohen's *h* corrects for this asymmetry. Raw difference gives a slightly wrong sample size.

### Why `hashlib.md5` for group assignment?
Python's built-in `hash()` is randomised across kernel restarts (Python 3.3+). If a returning user is re-assigned mid-experiment, results are contaminated. `hashlib.md5` is deterministic across runs, machines, and Python versions — exactly how Optimizely and Statsig work under the hood.

### Why full UUIDs (`SIA-<uuid>`) instead of sequential integers?
Real user IDs are alphanumeric strings, not clean integers. A naive `user_id % 2` split on sequential integers would perfectly alternate groups — that is not how real traffic arrives. Hashing a UUID breaks any underlying pattern before splitting.

### Why `numpy.random.default_rng` instead of `np.random.seed`?
`default_rng` creates an isolated Generator object using PCG64 — statistically superior to the older Mersenne Twister and fully reproducible via explicit seed. The legacy global seed can be silently corrupted by third-party library imports.

### Why 14 days minimum?
A "Limited seats" banner inflates conversion in Week 1 simply because it is novel. Two full weeks ensures the novelty effect washes out and the measured lift reflects true steady-state behaviour.

### Why upgrade from 80% to 90% power?
The power curve confirms 80% is the industry standard and sufficient. But at 10,000 daily sessions, 90% power costs only a few extra days. Missing a genuine lift at SIA's scale is commercially more expensive than running slightly longer — so we take it.

### Why both statistical AND practical significance gates?
A p-value of 0.049 on a 0.001pp lift is statistically significant but commercially worthless. The decision gate requires the observed relative lift to also meet the minimum threshold set by the revenue team.

---

## ✅ Five Randomisation Checks

Before touching any conversion data, the notebook verifies the simulation is fair across five dimensions:

| Check | Method | Why It Matters |
|---|---|---|
| **Group Size Balance** | Chi-square (SRM) | A mismatch here invalidates everything downstream |
| **Session Duration Balance** | T-test | Pre-existing engagement differences contaminate the lift |
| **Device Type Balance** | Chi-square | Mobile and desktop behave differently in booking funnels |
| **Day-of-Week Balance** | Chi-square | Weekend traffic behaves differently from weekday traffic |
| **User Uniqueness** | `nunique()` vs `len()` | Duplicate IDs inflate sample size and break t-test assumptions |

---

## ⚠️ Production Caveats

1. **Novelty effect** — Monitor conversion weekly post-launch. If lift decays after 4 weeks, the effect is novelty-driven, not genuine urgency response.
2. **Segmentation** — Validate the effect holds across route types (short-haul vs long-haul) and booking windows (same-day vs advance).
3. **No peeking** — This result is valid only because it was read once, at the predetermined sample size. Re-running and cherry-picking a favourable window inflates false positive rate.
4. **Inventory accuracy** — The banner must only fire when seats are genuinely limited. False scarcity is a brand risk for SIA. Losing customer trust costs far more than any 1.5pp conversion lift.

---

## 📦 Requirements

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

## 🚀 Running the Notebook

```bash
# Clone the repo
git clone https://github.com/<popolome>/SIA-Conversion-AB-Test.git
cd SIA-Conversion-AB-Test

# Install dependencies
pip install -r requirements.txt

# Launch Jupyter
jupyter notebook sia_conversion_ab_test.ipynb
```

Or open directly in **Google Colab** using the badge at the top of this README.

> ⚠️ **Run cells in order.** Each cell depends on variables set in earlier cells. Use `Kernel > Restart & Run All` for a clean run.

---

## 📊 Results (Simulation Output)

| Metric | Control | Treatment |
|---|---|---|
| Visitors | ~5,000 | ~5,000 |
| Conversion Rate | ~4.98% | ~6.69% |
| Relative Lift | — | ~34% |
| P-value | — | 0.0003 |
| 95% CI on difference | — | (0.0079, 0.0262) |

> Results vary slightly per run due to simulation randomness, but always exceed the 1.5pp MDE at the 90% power level.

---

## 👤 Author

**Your Name**

- GitHub: [@your-username](https://github.com/popolome)
- LinkedIn: [Your Name](https://www.linkedin.com/in/jun-kit-mak-611b4b108/)

---

## 📄 License

MIT License — see [LICENSE](LICENSE) for details.

---

**⭐ If you found this useful, consider starring the repo!**
