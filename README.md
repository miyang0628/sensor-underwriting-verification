# A Sensor-Data-Based Underwriting Verification Framework for Independent-Living Older Adults

Reproducibility repository for the manuscript

> **"A Sensor-Data-Based Underwriting Verification Framework for Independent-Living
> Older Adults: Combining PCMCI and DiCE"**
> *Anonymous author(s). Manuscript under double-blind review at an anonymous journal.*

This repository is released for **double-blind peer review**. It contains no
author, affiliation, or other identifying information. Any such information will
be added only in the camera-ready version.

---

## 1. What this repository is

The study does **not** propose a ready-to-deploy risk-prediction model. It designs
and demonstrates a **five-step verification framework** that an insurer would run
*before* deciding whether in-home sensor time series should enter the underwriting
process at all. The framework is a sequence of **gates**: a risk factor that fails a
step does not pass to the next, and the *reason it was excluded* is recorded as an
output. The contribution is the **structure of the procedure**, not the particular
numbers, which come from a 25-household proof of concept and are not meant to
generalize.

The five steps (and the review-stage concerns each addresses):

| Step | Concern addressed | Method |
|------|-------------------|--------|
| 1. Sample eligibility | reliability of a small/heterogeneous sample | unit redefined household → session; facility track separated |
| 2. Label validity | opaque composite label | reason text parsed into five abnormal categories |
| 3. Causal stability | correlation vs causation | PCMCI+ lag-1 discovery + A/B and leave-one-household-out stability |
| 4. Intervention validity | explanations without real leverage | DiCE counterfactuals constrained to causal paths + dual reliability gate |
| 5. Governance | scope of human oversight | two-dimensional HITL grid + fairness audit + documentation mapping |

The framework's effectiveness is shown not by the factors it passes but by the
factors it **excludes with a traceable reason** — made explicit here through a
gate-ablation study (`06`) and an unconstrained-counterfactual bypass analysis (`05`).

## 2. Repository layout

```
.
├── data/                     # input data placed here by the user (not redistributed)
├── notebooks/
│   ├── 00_data_preparation.ipynb   # Steps 1-2: parsing, sessions, facility split
│   ├── 01_eda.ipynb                # descriptive summary, class imbalance
│   ├── 02_pcmci.ipynb              # Step 3: PCMCI+ causal discovery
│   ├── 03_causal_stability.ipynb   # Step 3: A/B + leave-one-household-out stability
│   ├── 04_dice_intervention.ipynb  # Step 4: dual reliability gate + constrained DiCE
│   ├── 05_dice_bypass.ipynb        # unconstrained-DiCE causal-path bypass (quantified)
│   ├── 06_gate_ablation.ipynb      # utility: what each removed gate admits
│   ├── 07_hitl_governance.ipynb    # Step 5: 2-D HITL grid, fairness audit, case report
│   └── 08_robustness.ipynb         # reproduction vs sample size; bypass vs causal-path width; pilot-scale HITL downgrade
├── results/
│   ├── figures/              # PNG + PDF, grayscale, 600 dpi
│   └── tables/               # CSV
├── requirements.txt
└── README.md
```

Each notebook is **self-contained in a single code cell**, begins with an English
header comment, writes its figures (PNG and PDF, grayscale, 600 dpi) to
`results/figures/` and its tables to `results/tables/`, and prints no absolute file
paths.

## 3. Data

The analysis uses the **AI-Hub "Risk-Detection Data for the Care of Independent-Living
Older Adults"** dataset (multimodal home-sensor time series). The dataset is
distributed by AI-Hub under its own terms and is **not redistributed here**.

To reproduce, obtain the dataset from AI-Hub and place the prepared
observation-level table at:

```
data/raw_sessions.pkl
```

a pandas DataFrame with one row per 10-minute observation and columns for household
id, timestamp, demographics, the environment / vital-sign / emergency sensor
channels, and the integrated judgement label and its reason text. Notebook `00`
derives every downstream file from it.

> **Note.** `data/` is git-ignored so that no restricted data is committed. Only code
> and derived `results/` are intended for version control.

## 4. How to run

```bash
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace notebooks/00_data_preparation.ipynb
# then 01 → 08 in order (each reads artifacts written by earlier notebooks)
```

Run the notebooks **in numerical order**: `00` writes the analysis-ready datasets and
a variable dictionary, `02`–`03` write the causal-path and reproduction artifacts,
`04`–`07` consume them, and `08` runs the additional robustness checks (reproduction
stability as a function of sample size, counterfactual bypass as a function of
causal-path width, and the pilot-scale HITL grade downgrade). Step 3 runs PCMCI+ per household twice (two significance
levels) and caches the per-household results under `data/_pcmci_cache_*.pkl`, so
re-runs are fast.

## 5. Reproducibility notes

- All stochastic steps use a fixed seed.
- The partial-correlation magnitudes recovered by PCMCI+ are, as expected for a
  single-lag linear conditional dependence at a 10-minute cadence, **weak**; only the
  **sign and rank** of links are intended to reproduce, which is what the stability
  checks in `03` assess. The A/B check is an **internal** resampling of one sample,
  **not** an independent external replication — the latter remains future work.
- Numbers are a 25-household proof of concept and are not claims of population-level
  effect sizes.

## 6. License

Code: released for review purposes. A license will be specified in the camera-ready
version. The AI-Hub dataset is governed by its own terms and is not included.
