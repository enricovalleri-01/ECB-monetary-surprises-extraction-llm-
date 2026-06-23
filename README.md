# ECB Communications as Monetary Shocks
### A Multi-Agent LLM Framework and Asymmetric Transmission Analysis

**Master's Thesis** | Enrico Valleri | Alma Mater Studiorum — Università di Bologna  
**Supervisor:** Prof. Umberto Cherubini | Department of Economics  
M.Sc. in Applied Economics and Markets | A.Y. 2025/2026

---

## Overview

This repository contains the code and data for the master's thesis *"ECB Communications as Monetary Shocks: A Multi-Agent LLM Framework and Asymmetric Transmission Analysis"*.

The project pursues two connected objectives. The first is methodological: constructing a historical series of monetary policy surprises for the European Central Bank (ECB) over the period March 1999 – December 2025, covering all 312 Governing Council meetings, by extracting quantitative signals directly from the ECB's official communications using a four-stage pipeline. The second is empirical: using this surprise series as a monetary policy shock in a Local Projections framework to estimate the dynamic transmission of ECB surprises on euro area HICP inflation, testing for asymmetry between hawkish and dovish shocks.

The pipeline is built around two methodological principles:

- **Ex-ante construction**: every document used by the pipeline was publicly available before the ten-day pre-meeting blackout period. This applies to the document inputs; the language model reading them was trained on more recent data, a residual limitation acknowledged in the thesis.
- **Market neutrality**: expectations are formed exclusively from official institutional communications, without incorporating any asset price information.

---

## Repository Structure

```
ecb-monetary-surprises-llm/
│
├── notebooks/
│   ├── ECB_Pipeline_MASTER.ipynb        # Main LLM pipeline (4 stages)
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

The pipeline consists of four sequential stages: two LLM agents that read documents (Agents IM and IB) and two deterministic modules that synthesise their outputs (Agents II and III). Only Agents IM and IB call the Gemini API; Agents II and III are fully arithmetic.

| Stage | Role | Input | Output |
|-------|------|-------|--------|
| **Agent IM** | Monetary Intelligence | ECB Accounts (2015–2025) or Press Conference Introductory Statement (1999–2014) | Hawkishness score, forward guidance classification (Odyssean/Delphic), prior probability distribution over rate bins |
| **Agent IB** | Economic Conditions | Economic/Monthly Bulletin + Governing Council speeches (28-day window, 10-day blackout) | Five economic condition scores; aggregate index s_IB = 0.50·e₁ + 0.30·e₂ + 0.20·e₃ |
| **Agent II** | Synthesis (deterministic) | Outputs of IM and IB | Posterior probability distribution via two-stage softmax update: s_policy = 0.60·h_IM + 0.20·s_FG + 0.20·e₄; s_econ = 0.70·s_IB + 0.30·e_speech; s̃ = 0.55·s_policy + 0.45·s_econ |
| **Agent III** | Surprise Calculation (deterministic) | Agent II posterior + actual ECB decision | Mechanical surprise s_mech = Δ_actual − E[Δ]; salience and normalised measures |

All LLM calls use Google Gemini 2.5 Flash at temperature 0.1.

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
| Meetings | 312 | 187 | 76 | 49 |
| Mean | −1.1 bp | −5.5 bp | +6.5 bp | +3.8 bp |
| Std dev | 13.5 bp | 13.3 bp | 9.3 bp | 13.3 bp |
| No surprise | 135 (43%) | 78 (42%) | 31 (41%) | 26 (53%) |
| Hawkish | 79 (25%) | 25 (13%) | 39 (51%) | 15 (31%) |
| Dovish | 98 (31%) | 84 (45%) | 6 (8%) | 8 (16%) |

Presidential eras: Duisenberg/Trichet March 1999 – October 2011; Draghi November 2011 – October 2019; Lagarde November 2019 – December 2025.

Note on the Draghi era: the 51% hawkish rate reflects the ZLB mechanism — the pipeline frequently assigned probability to cuts that did not materialise, generating small positive (hawkish) surprises at hold meetings. The standard deviation of 9.3 bp is the lowest of the three eras.

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

**Document coverage note**: the 312 meetings include ~45 bi-weekly sessions (1999–2001) and unscheduled sessions that did not have a dedicated press conference. Those meetings reuse the Introductory Statement of the most recent monthly conference; this explains the gap between 267 press conference transcripts and 312 total meetings. The document substitution from Introductory Statements (pre-2015) to Accounts (post-2015) is assessed formally via an F-test for equality of means (F = 5.135, p = 0.006).

---

## Empirical Results

The surprise series is used as a monetary policy shock in a Local Projections framework (Jordà, 2005) to estimate the dynamic transmission of ECB surprises on euro area HICP inflation.

### Main Finding — Asymmetric Transmission

Shocks are normalised to ±25 bp. At horizon h = 9 months:

| | Coefficient | Std error |
|-|-------------|-----------|
| Hawkish surprise (β⁺) | −0.857 pp | (0.302) |
| Dovish surprise (β⁻) | +0.480 pp | (0.346) |
| Wald test p-value | | 0.009 |

The asymmetry is statistically significant at 14 out of 24 estimated horizons, concentrated in the h = 1 to h = 16 window — a ratio of approximately 1.8 to 1.

**Key robustness**: excluding the 2022–2023 hiking cycle, the hawkish coefficient *increases* to −1.103 pp (Wald p = 0.028), widening the ratio to ~4.5 to 1. The asymmetry is a structural feature of the sample, not an artefact of the 2022 episode.

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

## Citation

```
Valleri, E. (2026). ECB Communications as Monetary Shocks: A Multi-Agent LLM Framework
and Asymmetric Transmission Analysis. Master's Thesis, Università di Bologna.
```
