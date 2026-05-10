# ECB-monetary-surprises-extraction-llm-

# ECB Communications as Monetary Shocks: A Multi-Agent LLM Framework and Asymmetric Transmission Analysis

> **Master's Thesis** | Enrico Valleri | Alma Mater Studiorum — Università di Bologna  
> Supervisor: Prof. Umberto Cherubini | Department of Economics | M.Sc. in Applied Economics and Markets | A.Y. 2025/2026

---

## Overview

This repository contains the code and data for the master's thesis *"ECB Communications as Monetary Shocks: A Multi-Agent LLM Framework and Asymmetric Transmission Analysis"*.

The project pursues two connected objectives. The first is methodological: constructing a historical series of **monetary policy surprises** for the European Central Bank (ECB) over the period **March 1999 – December 2025**, covering all 312 Governing Council meetings, by extracting quantitative signals directly from the ECB's official communications using a sequence of four large language model agents. The second is empirical: using this surprise series as a monetary policy shock in a **Local Projections** framework to estimate the dynamic transmission of ECB surprises on euro area inflation, testing for asymmetry between hawkish and dovish shocks.

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
│   ├── ecb_policy_decisions_OFFICIAL.csv
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

The pipeline consists of four sequential agents, each processing a specific set of documents and producing a structured JSON output passed to the next agent. All agents use **Google Gemini 2.5 Flash** at temperature 0.1.

| Agent | Role | Input | Output |
|---|---|---|---|
| **Agent IM** | Monetary Intelligence | ECB Accounts (2015–2025) or Press Conference Introductory Statement (1999–2014) | Hawkishness score, council division, forward guidance classification, prior probability distribution |
| **Agent IB** | Economic Conditions | Economic/Monthly Bulletin + Governing Council speeches (28-day window, 10-day blackout) | Five economic condition scores weighted by ECB mandate hierarchy |
| **Agent II** | Synthesis | Outputs of IM and IB | Posterior probability distribution over rate decisions via softmax update |
| **Agent III** | Surprise Calculation | Agent II posterior + actual ECB decision | Mechanical, salience, and normalised surprise measures |

---

## Dataset

### Surprise Series (`data/surprises_timeseries.csv`)

312 Governing Council meetings, March 1999 – December 2025. Key columns:

| Column | Description |
|---|---|
| `meeting_date` | Date of the Governing Council meeting |
| `dfr_surprise_mech` | Mechanical surprise on the Deposit Facility Rate (bp) |
| `mro_surprise_mech` | Mechanical surprise on the Main Refinancing Operations rate (bp) |
| `mlf_surprise_mech` | Mechanical surprise on the Marginal Lending Facility rate (bp) |
| `dfr_direction` | Classification: `hawkish_surprise`, `dovish_surprise`, `no_surprise` |
| `expected_change_bp` | Model's expected rate change (bp) |
| `entropy` | Shannon entropy of the posterior distribution |
| `p_hold` | Posterior probability of no change |

### Summary Statistics (DFR mechanical surprise)

| | Full sample | Pre-2015 | Post-2015 |
|---|---|---|---|
| Meetings | 312 | 225 | 87 |
| Mean | −1.1 bp | −3.7 bp | +5.0 bp |
| Std dev | 13.5 bp | 13.2 bp | 8.6 bp |
| No surprise | 135 (43%) | — | — |
| Hawkish | 79 (25%) | — | — |
| Dovish | 98 (31%) | — | — |

### Document Sources

All documents are publicly available from the ECB website:

| Document | Period | Agent | Source |
|---|---|---|---|
| Accounts of the Monetary Policy Meeting | 2015–2025 | Agent IM | [ECB Accounts](https://www.ecb.europa.eu/press/accounts/) |
| Press Conference — Introductory Statement | 1999–2014 | Agent IM | [ECB Press Conferences](https://www.ecb.europa.eu/press/pressconf/) |
| Economic Bulletin | 2015–2025 | Agent IB | [ECB Economic Bulletin](https://www.ecb.europa.eu/pub/economic-bulletin/) |
| Monthly Bulletin | 1999–2014 | Agent IB | ECB website |
| Governing Council speeches | 1999–2025 | Agent IB | [ECB Speeches](https://www.ecb.europa.eu/press/key/) |

---

## Empirical Results

The surprise series is used as a monetary policy shock in a **Local Projections** framework (Jordà, 2005) to estimate the dynamic transmission of ECB surprises on euro area HICP inflation.

**Main finding — Asymmetric transmission:**

Hawkish and dovish surprises are estimated separately:

$$y_{t+h} - y_{t-1} = \alpha_h + \beta_h^+ s_t^+ + \beta_h^- s_t^- + \gamma_h X_t + \varepsilon_{t+h}$$

At horizon $h = 9$ months:
- **Hawkish surprise**: −0.857 pp (statistically significant)
- **Dovish surprise**: +0.480 pp
- **Wald test** for equality of coefficients: **p = 0.009**

The asymmetry is statistically significant across 14 consecutive horizons, indicating that monetary tightening surprises have substantially larger disinflationary effects than easing surprises of equal magnitude. Results are robust to the inclusion of a post-2015 dummy and alternative lag structures.

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
