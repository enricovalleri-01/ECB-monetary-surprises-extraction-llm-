# ECB Communications as Monetary Shocks
### A Multi-Agent LLM Framework and Asymmetric Transmission Analysis

**Master's Thesis** | Enrico Valleri | Alma Mater Studiorum — Università di Bologna  
**Supervisor:** Prof. Umberto Cherubini | Department of Economics  
M.Sc. in Applied Economics and Markets | A.Y. 2025/2026

---

## Overview

This repository contains the code and data for the master's thesis *"ECB Communications as Monetary Shocks: A Multi-Agent LLM Framework and Asymmetric Transmission Analysis"*.

The project pursues two connected objectives. The first is methodological: constructing a historical series of monetary policy surprises for the European Central Bank (ECB) over the period March 1999 – December 2025, covering all 312 Governing Council meetings, by extracting quantitative signals directly from the ECB's official communications using a sequence of four large language model agents. The second is empirical: using this surprise series as a monetary policy shock in a Local Projections framework to estimate the dynamic transmission of ECB surprises on euro area inflation, testing for asymmetry between hawkish and dovish shocks.

The pipeline is built around two methodological principles:

- **Ex-ante purity**: every document used by the agents was publicly available before the ten-day pre-meeting blackout period — no look-ahead bias by construction.
- **Market neutrality**: expectations are formed exclusively from official institutional communications, without incorporating any asset price information.

---

## Repository Structure

```
ecb-monetary-surprises-llm/
│
├── notebooks/
│   ├── ECB_Pipeline_MASTER.ipynb        # Main LLM pipeline (4 agents)
│   └── ECB_LocalProjections.ipynb       # Econometric analysis (Local Projections)
│
├── data/
│   ├── surprises_timeseries.csv         # Final surprise dataset (312 meetings)
│   ├── ecb_policy_decisions_FULL_1999_2025.csv
│   └── ecb_surprises_metadata.json
│
├── figures/
│   ├── lp_irfs_monthly.png              # Impulse response functions
│   └── lp_asymmetric_wald.png           # Asymmetry test results
│
└── README.md
```

---

## Pipeline Architecture

The pipeline consists of four sequential agents, each processing a specific set of documents and producing a structured JSON output passed to the next agent. All agents use Google Gemini 2.5 Flash at temperature 0.1.

| Agent | Role | Input | Output |
|-------|------|-------|--------|
| **Agent IM** | Monetary Intelligence | ECB Accounts (2015–2025) or Press Conference Introductory Statement (1999–2014) | Hawkishness score, council division, forward guidance classification, prior probability distribution |
| **Agent IB** | Economic Conditions | Economic/Monthly Bulletin + Governing Council speeches (28-day window, 10-day blackout) | Five economic condition scores weighted by ECB mandate hierarchy |
| **Agent II** | Synthesis | Outputs of IM and IB | Posterior probability distribution over rate decisions via softmax update |
| **Agent III** | Surprise Calculation | Agent II posterior + actual ECB decision | Mechanical, salience, and normalised surprise measures |

---

## Dataset

### Surprise Series (`data/surprises_timeseries.csv`)

312 Governing Council meetings, March 1999 – December 2025. Key columns:

| Column | Description |
|--------|-------------|
| `meeting_date` | Date of the Governing Council meeting |
| `dfr_surprise_mech` | Mechanical surprise on the Deposit Facility Rate (bp) |
| `mro_surprise_mech` | Mechanical surprise on the Main Refinancing Operations rate (bp) |
| `mlf_surprise_mech` | Mechanical surprise on the Marginal Lending Facility rate (bp) |
| `dfr_direction` | Classification: `hawkish_surprise`, `dovish_surprise`, `no_surprise` |
| `expected_change_bp` | Model's expected rate change (bp) |
| `entropy` | Shannon entropy of the posterior distribution |
| `p_hold` | Posterior probability of no change |

### Summary Statistics (DFR mechanical surprise)

|  | Full sample | Duisenberg/Trichet | Draghi | Lagarde |
|--|------------|-------------------|--------|---------|
| Meetings | 312 | 189 | 75 | 48 |
| Mean | −1.3 bp | −5.6 bp | +6.4 bp | +3.7 bp |
| Std dev | 13.0 bp | 13.2 bp | 8.6 bp | 13.4 bp |
| No surprise | 135 (43%) | 67 (35%) | 46 (61%) | 22 (46%) |
| Hawkish | 79 (25%) | 41 (22%) | 21 (28%) | 17 (35%) |
| Dovish | 98 (31%) | 81 (43%) | 8 (11%) | 9 (19%) |

Presidential eras: Duisenberg/Trichet March 1999 – October 2011; Draghi November 2011 – October 2019; Lagarde November 2019 – December 2025.

---

## Document Sources

All documents are publicly available from the ECB website:

| Document | Period | Agent | Source |
|----------|--------|-------|--------|
| Accounts of the Monetary Policy Meeting | 2015–2025 | Agent IM | [ECB Accounts](https://www.ecb.europa.eu/press/accounts/html/index.en.html) |
| Press Conference — Introductory Statement | 1999–2014 | Agent IM | [ECB Press Conferences](https://www.ecb.europa.eu/press/pressconf/html/index.en.html) |
| Economic Bulletin | 2015–2025 | Agent IB | [ECB Economic Bulletin](https://www.ecb.europa.eu/pub/economic-bulletin/html/index.en.html) |
| Monthly Bulletin | 1999–2014 | Agent IB | ECB website |
| Governing Council speeches | 1999–2025 | Agent IB | [ECB Speeches](https://www.ecb.europa.eu/press/key/html/index.en.html) |

A note on document availability: the ECB introduced the Accounts of the Governing Council only in January 2015. For the preceding period (1999–2014), Agent IM uses the Introductory Statement of the Press Conference of the prior meeting as a substitute. The statistical consequences of this document substitution are assessed formally via an F-test for equality of means (F = 5.135, p = 0.006).

---

## Empirical Results

The surprise series is used as a monetary policy shock in a Local Projections framework (Jordà, 2005) to estimate the dynamic transmission of ECB surprises on euro area HICP inflation.

### Main Finding — Asymmetric Transmission

Hawkish and dovish surprises are estimated separately. At horizon h = 9 months:

| | Coefficient | Std error |
|-|-------------|-----------|
| Hawkish surprise | −0.857 pp | (0.302) |
| Dovish surprise | +0.480 pp | (0.346) |
| Wald test p-value | | 0.009 |

The asymmetry is statistically significant at 14 out of 24 estimated horizons, concentrated in the h = 1 to h = 16 window — a ratio of approximately 1.8 to 1. Results are robust to the inclusion of a post-2015 dummy and alternative agent weighting schemes.

---

## Requirements

```
google-generativeai
pydantic
pandas
numpy
statsmodels
scipy
matplotlib
seaborn
pdfplumber
beautifulsoup4
requests
tqdm
```

Install with:

```bash
pip install google-generativeai pydantic pandas numpy statsmodels scipy matplotlib seaborn pdfplumber beautifulsoup4 requests tqdm
```

---

## Usage

1. Clone the repository
2. Mount your Google Drive in Colab and update the path in `Config` (Cell 1 of `ECB_Pipeline_MASTER.ipynb`)
3. Add your Google AI API key:
   ```python
   GEMINI_API_KEY = 'your_api_key_here'
   ```
4. Run `ECB_Pipeline_MASTER.ipynb` to process meetings and generate surprises — the pipeline saves checkpoint files after each meeting and resumes automatically if interrupted
5. Run `ECB_LocalProjections.ipynb` for the econometric analysis

---

