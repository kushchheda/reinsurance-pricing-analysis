# French Motor TPL: Reinsurance Pricing Analysis

A portfolio project demonstrating an end-to-end reinsurance pricing workflow on real insurance data.

---

## What This Project Achieves

This data science portfolio project covers not only prediction, but goes further by translating model outputs into a real business decision: **how much should a reinsurer charge to cover the tail risk of an insurance portfolio?**

Using 668,874 real French motor insurance policies, this project builds a complete pricing pipeline.

- It identifies which policyholders drive the most risk, using the same risk factors (driver age, Bonus-Malus score, vehicle type) that underwriters rely on when assessing a book of business.
- It produces a **Pure Premium** for every policy, the expected annual loss that forms the foundation of all non-life insurance pricing.
- It prices a **per-risk Excess of Loss (XL) treaty** using the industry-standard lognormal Limited Expected Value method, outputting Layer Loss Cost and Loss-on-Line by risk segment.

The key finding: the top 25% of highest-risk policies account for 75.6% of total reinsurance loss cost, this concentration is precisely why XL reinsurance exists.

---

## Dataset

**freMTPL2** French Motor Third-Party Liability insurance data.
Source: Charpentier (2014), *Computational Actuarial Science with R*.

| File | Rows | Description |
|------|------|-------------|
| `freMTPL2freq.csv` | 678,013 | One row per policy with risk features and claim count |
| `freMTPL2sev.csv` | 26,639 | One row per claim with amount paid |

Key fields: `Exposure` (fraction of year active), `ClaimNb` (claims filed), `BonusMalus` (French no-claims discount score), `VehBrand`, `DrivAge`, `Area`, `Density`

Data available at: kaggle.com/datasets/karansarpal/fremtpl2-french-motor-tpl-insurance-claims

---

## Project Structure

```
reinsurance_pricing_analysis/
├── reinsurance_pricing_analysis.ipynb
├── freMTPL2freq.csv
├── freMTPL2sev.csv
├── figures/
│   ├── task1_eda.png
│   ├── task2_glm.png
│   └── task3_xl_pricing.png
└── README.md
```

---

## The Three Tasks

### Task 1: Exploratory Data Analysis

Before any modelling, we establish which risk factors drive claims in this portfolio:

- Claim rate by driver age band and Bonus-Malus score
- Claim severity distribution by vehicle brand
- Distributional analysis of claim amounts

**Finding:** Young drivers (18-22) have the highest claim frequency. Drivers with a Bonus-Malus score above 100 claim at 3-4 times the rate of safe drivers. Claim amounts are heavily right-skewed, with a mean nearly double the median. This skew is the economic justification for reinsurance.

### Task 2: Frequency and Severity GLM Modelling

Industry-standard two-part actuarial model:

- **Poisson GLM** for claim frequency, with `log(Exposure)` as an offset so predictions are expressed as claims per policy-year regardless of how long each policy was active
- **Gamma GLM** for claim severity, fitted only on policies that had at least one claim
- Combined into **Pure Premium = Predicted Frequency x Predicted Severity**

**Finding:** The lift chart confirms the model correctly separates high-risk from low-risk policies. The top decile has materially higher actual claim frequency than the bottom decile, meaning the model produces actionable risk rankings. Mean pure premium across the portfolio is approximately 194 euros per policy-year.

### Task 3: XL Reinsurance Treaty Pricing

Price a per-risk Excess of Loss treaty using actuarial loss cost methods:

- **Treaty structure:** 400,000 euros xs 100,000 euros. The cedant retains the first 100k of each claim; The reinsurer covers the next 400k (the 100k to 500k layer).
- **Method:** Lognormal Limited Expected Value (LEV) formula computes the expected loss falling within the layer analytically, for each policy's predicted severity.
- **Outputs:** Layer Loss Cost and Loss-on-Line per policy and by risk segment.

**Finding:** The High Risk tier (25% of policies) drives 75.6% of total reinsurance loss cost. The Loss Concentration Curve shows a steep tail, confirming that XL pricing must be highly sensitive to the composition of the high-risk segment. A reinsurer that misprices this tier will be systematically undercharged.

---

## Results Summary

| Metric | Value |
|--------|-------|
| Policies analysed | 668,874 |
| Overall claim rate | 3.73% |
| Poisson GLM Pseudo R-squared | 0.040 |
| Mean Pure Premium | 194 euros per policy-year |
| XL Treaty Layer | 400k xs 100k |
| Total Portfolio Layer Loss Cost | 586,166 euros |
| High Risk tier share of reinsurance losses | 75.6% |

---

## Setup and Running

```bash
# Create and activate a virtual environment
python -m venv venv
venv\Scripts\activate        # Windows
source venv/bin/activate     # Mac/Linux

# Install dependencies
pip install pandas numpy matplotlib seaborn statsmodels scipy jupyter

# Place freMTPL2freq.csv and freMTPL2sev.csv in the project folder
# Then launch the notebook
jupyter notebook reinsurance_pricing_analysis.ipynb
```

Run all cells in order. All figures will be saved to the `figures/` subfolder automatically.

---

## License

MIT License

Copyright (c) 2025

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
