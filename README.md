# pma-shared
Shared infrastructure, task definitions, and evaluation conventions for the duke-pma-analytics organization. This repository establishes the canonical task definitions and evaluation framework used across all projects in the lab ensuring that results are reproducible and comparable across studies.

## Foundational Reference
 
All task definitions in this repository are grounded in:
 
> Hill, E. D., Kashyap, P., Raffanello, E., Wang, Y., Moffitt, T. E., Caspi,
> A., Engelhard, M., & Posner, J. (2025). Prediction of mental health risk in
> adolescents. *Nature Medicine*. https://doi.org/10.1038/s41591-025-03560-7
 
The Hill et al. framework trained neural network models on ABCD Study data
(n > 11,000) to predict p-factor quartile transitions, evaluating three
clinically distinct scenarios: **conversion**, **persistence**, and
**agnostic** prediction. This repository adopts those definitions verbatim as
the standard evaluation protocol for all PMA lab projects using ABCD data.
 
---
 
## Task Definitions
 
The **p-factor** is derived from single-factor exploratory factor analysis on
8 CBCL syndrome scales (anxious/depressed, withdrawn/depressed, somatic
complaints, social problems, thought problems, attention problems,
rule-breaking behavior, aggressive behavior), then binned into quartiles:
 
| Quartile | Label       |
|----------|-------------|
| Q1       | No-risk     |
| Q2       | Low-risk    |
| Q3       | Moderate-risk |
| Q4       | High-risk   |
 
### Task 1: Next-Quartile Prediction (Agnostic)
**Definition:** Predict which p-factor quartile a participant will be in at
the next follow-up event, from any prior quartile.
 
- **Input:** All predictors at events T, T−1, ..., T−k
- **Output:** P-factor quartile at event T+1
- **Metric:** Macro AUROC (4-class, one-vs-rest)
- **Exclusion:** Participants with no valid p-factor at any follow-up event;
  predictors from the 4-year follow-up (no subsequent event available)
- **Hill et al. reference:** AUROC = 0.80 ± 0.01 (questionnaire model),
  0.89 ± 0.01 (CBCL scales model)
 
This is the **primary training objective** used in ATLAS Phase 1.
 
---
 
### Task 2: Conversion
**Definition:** Among participants currently in Q1–Q3 (not high-risk),
predict who will move into Q4 (high-risk) at the next event.
 
- **Input:** All predictors at T and prior events, restricted to participants
  with p-factor in Q1, Q2, or Q3 at T
- **Output:** Binary — does participant enter Q4 at T+1?
- **Metric:** AUROC (binary)
- **Exclusion:** Participants already in Q4 at time T; participants with no
  valid p-factor at T+1
- **Prevalence:** ~11% of transitions in the ABCD cohort (Hill et al.)
- **Hill et al. reference:** AUROC = 0.75 ± 0.01 (questionnaire model),
  0.84 ± 0.01 (CBCL scales model)
- **Clinical motivation:** Identifying children whose mental health is
  deteriorating enables targeted preventive intervention — the most
  actionable prediction scenario
 
---
 
### Task 3: Persistence
**Definition:** Among participants currently in Q4 (high-risk), predict who
will remain in Q4 at the next event.
 
- **Input:** All predictors at T and prior events, restricted to participants
  with p-factor in Q4 at T
- **Output:** Binary — does participant remain in Q4 at T+1?
- **Metric:** AUROC (binary)
- **Exclusion:** Participants not in Q4 at time T; participants with no valid
  p-factor at T+1
- **Prevalence:** ~66% of high-risk participants persist (Hill et al.)
- **Hill et al. reference:** AUROC = 0.69 ± 0.02 (questionnaire model),
  0.78 ± 0.01 (CBCL scales model)
- **Clinical motivation:** Predicting persistent illness informs treatment
  intensity and resource allocation
 
---
 
### Task 4: Agnostic High-Risk Prediction
**Definition:** Predict whether a participant will be in Q4 at the next event,
regardless of their current quartile.
 
- **Input:** All predictors at T and prior events
- **Output:** Binary — is participant in Q4 at T+1?
- **Metric:** AUROC (binary)
- **Exclusion:** Participants with no valid p-factor at T+1
- **Hill et al. reference:** AUROC = 0.80 ± 0.01 (questionnaire model),
  0.89 ± 0.01 (CBCL scales model)
- **Note:** This is the broadest evaluation and subsumes both conversion and
  persistence; it is the ceiling for the other two tasks
 
---
 
## Evaluation Conventions
 
### Standard Splits
Follow Hill et al.: 8:1:1 train/val/test split, stratified by participant
(not by event) to prevent data leakage across longitudinal observations.
All learnable preprocessing (z-score scaling, mean imputation) fit on
training set only.
 
### Metrics
- **Primary:** Macro AUROC for Task 1; binary AUROC for Tasks 2–4
- **Secondary:** Average Precision (area under precision-recall curve)
- **Confidence intervals:** Bootstrap resampling (n=1,000) of test set predictions
- **Subgroup evaluation:** Stratify by sex, race/ethnicity, ADI quartile,
  age, follow-up event, and event year (see Hill et al. Table 3)
 
### Reporting Template
All projects should report the following comparison table when evaluating
against Hill et al.:
 
| Scenario    | Model          | AUROC | Hill Questionnaire | Hill CBCL |
|-------------|----------------|-------|--------------------|-----------|
| Conversion  | [your model]   | —     | 0.75 ± 0.01        | 0.84 ± 0.01 |
| Persistence | [your model]   | —     | 0.69 ± 0.02        | 0.78 ± 0.01 |
| Agnostic    | [your model]   | —     | 0.80 ± 0.01        | 0.89 ± 0.01 |
 
The Hill et al. questionnaire model is the **mechanism-driven baseline**
(no CBCL symptom features). The CBCL model is the **symptom-driven ceiling**.
Projects should position themselves relative to both.
 
---
 
## P-Factor Construction
 
```python
from sklearn.decomposition import FactorAnalysis
import pandas as pd
 
CBCL_SYNDROME_SCALES = [
    "cbcl_scr_syn_anxdep_r",      # Anxious/Depressed
    "cbcl_scr_syn_withdep_r",     # Withdrawn/Depressed
    "cbcl_scr_syn_somatic_r",     # Somatic Complaints
    "cbcl_scr_syn_social_r",      # Social Problems
    "cbcl_scr_syn_thought_r",     # Thought Problems
    "cbcl_scr_syn_attention_r",   # Attention Problems
    "cbcl_scr_syn_rulebreak_r",   # Rule-Breaking Behavior
    "cbcl_scr_syn_aggressive_r",  # Aggressive Behavior
]
 
def compute_pfactor(df: pd.DataFrame) -> pd.Series:
    """
    Single-factor EFA on CBCL syndrome scales -> p-factor scores.
    Consistent with Hill et al. (2025) and Clark et al. (2021).
    """
    fa = FactorAnalysis(n_components=1, random_state=42)
    scores = fa.fit_transform(df[CBCL_SYNDROME_SCALES].dropna())
    return pd.Series(scores.squeeze(), index=df.dropna(subset=CBCL_SYNDROME_SCALES).index)
 
def quartile_labels(pfactor: pd.Series) -> pd.Series:
    """Bin p-factor into quartiles: 0=no-risk, 1=low, 2=moderate, 3=high."""
    return pd.qcut(pfactor, q=4, labels=[0, 1, 2, 3]).astype(int)
```
 
---
 
## Repository Structure
 
```
pma-shared/
  tasks/
    definitions.py        # Task filters, exclusion criteria, label construction
    metrics.py            # AUROC, AP, bootstrap CI, subgroup eval
    pfactor.py            # P-factor computation and quartile binning
  data/
    preprocessing.py      # Z-score, imputation, train/val/test split utils
    abcd_loader.py        # ABCD Study data loading conventions
  evaluation/
    compare_hill.py       # Standardized comparison table vs Hill et al. baselines
    subgroup_eval.py      # Stratified evaluation by demographic subgroups
  configs/
    hill_baselines.yaml   # Hill et al. reported numbers for reference
  docs/
    task_definitions.md   # This document (human-readable)
    hill_2025_summary.md  # Key findings from Hill et al. relevant to all projects
```
 
---
 
## Projects Using This Framework
 
| Repository | Description | Tasks Used |
|------------|-------------|------------|
| [atlas](https://github.com/duke-pma-analytics/atlas) | Self-supervised transformer for transdiagnostic structure discovery (ABCD) | All 4 |
| [evigen](https://github.com/duke-pma-analytics/evigen) | Evidence-guided clinical report generation with faithfulness PRMs | — |
| [iris](https://github.com/duke-pma-analytics/iris) | Clinical evidence retrieval | — |
 
---
 
## Notes on Conversion and Persistence Exclusions
 
The exclusion criteria for conversion and persistence tasks are mechanically
necessary, not a modeling choice:
 
- **Conversion** excludes Q4 participants at time T because by definition
  they cannot convert (they are already high-risk). Evaluating conversion
  on Q4 participants would conflate it with persistence.
- **Persistence** excludes Q1–Q3 participants at time T because by definition
  they cannot persist in Q4 (they were never in Q4). This is not a data
  quality issue — it is a task definition requirement.
 
These exclusions reduce the evaluation set size but are required to maintain
the clinical validity of each scenario as defined in Hill et al.
 
---
 
## Citation
 
If you use these task definitions in published work, please cite:
 
```bibtex
@article{hill2025prediction,
  title={Prediction of mental health risk in adolescents},
  author={Hill, Elliot D and Kashyap, Pratik and Raffanello, Elizabeth and
          Wang, Yun and Moffitt, Terrie E and Caspi, Avshalom and
          Engelhard, Matthew and Posner, Jonathan},
  journal={Nature Medicine},
  year={2025},
  doi={10.1038/s41591-025-03560-7}
}
```
