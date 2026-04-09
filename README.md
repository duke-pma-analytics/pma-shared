# pma-shared
Shared infrastructure, task definitions, and evaluation conventions for the duke-pma-analytics organization. This repository establishes the canonical task definitions and evaluation framework used across all projects in the lab ensuring that results are reproducible and comparable across studies.

## Project Description
 
This study, in response to RFA-MH-25-195, aims to enhance, deploy, and rigorously validate the Duke Predictive Model of Adolescent Mental Health (Duke-PMA), which uses a novel clinical signature derived from affordable, accessible measures to identify youth at high risk for psychiatric illness in primary care settings. The Duke-PMA, a neural network-based predictive tool, has already demonstrated high accuracy in predicting psychiatric risk one year in advance in youth aged 10-15, using data from the Adolescent Brain and Cognitive Development (ABCD) study. Notably, sleep disturbances have emerged as a key modifiable predictor in the model. Unlike most predictive models that rely on current symptoms to anticipate outcomes, the Duke-PMA bases its predictions on underlying disease mechanisms and protective factors, making it better suited to inform preventive interventions. Furthermore, the model identifies an elevated p-factor, a general measure of psychopathology that spans multiple psychiatric conditions, making it broadly applicable across diverse youth populations. Our project will begin by optimizing the Duke-PMA through the incorporation of behavioral tasks from the NIH Toolbox to enhance its prediction performance. Following Duke AI Health’s Algorithm-Based Clinical Decision Support Oversight framework, we will ensure the model adheres to the highest standards of transparency, quality, and equity. Additionally, we will apply trustworthy AI techniques designed to reduce effects of distribution shifts on model performance to ensure the model remains effective and equitable across diverse clinical settings and demographic groups. After optimization, the Duke-PMA will be deployed in rural primary care and pediatric clinics, where access to mental health services and research participation is often limited. We will enroll 2,000 youth from rural clinics in the Southeast and Midwest, partnering with the Science, Technology, and Research (STAR) Clinical Research Network. We will also explore the benefit of adding a measure of home environments to the Duke-PMA through digital envirotyping, which uses an AI-driven approach to assess home environments remotely without requiring in-person visits, making it much more resource-efficient and accessible than current approaches. Model performance will be validated through psychiatric diagnostics conducted one year after the initial assessment. If successful, this project has the potential to transform mental health resource allocation particularly in underserved communities by offering an accessible, low-cost, data-driven approach to identify vulnerable youth and highlight modifiable risk factors for early intervention.

Aim 1:
To prepare the Duke Predictive Model of Adolescent Mental Health (Duke-PMA) for clinical use by optimizing portability, actionability and implementation feasibility, and performance, leveraging Duke AI Health’s Algorithm-Based Clinical Decision Support Oversight to ensure the highest standards of transparency, quality, and equity. H1: Portability. Leveraging the ample site and demographic variation of the ABCD study, adaptations of Duke-PMA will prioritize predictive features and methods that are robust to variations in population demographics and data collection practices, thus ensuring health equity and high performance across clinical settings. H2: Actionability and implementation feasibility will be optimized by selecting model features that maximize performance yet are actionable and minimize implementation barriers (e.g., avoids measures poorly suited for rural settings as determined by our Community Advisory Board). H3: Performance. Continuing our work with the ABCD study, integrating NIH Toolbox (behavioral) measures will augment Duke-PMA’s predictive performance.

Aim 2:
To deploy and prospectively validate Duke-PMA (refined in Aim 1) in rural, primary care clinical settings. We will develop a tablet-based platform through which youth and their parents can complete questionnaires and behavioral tasks to inform our predictive model. Youth (n=2,000 youth; ages 10-15) and a parent will be recruited from rural pediatric primary care clinics to complete our assessment and will be contacted again 1 year later to assess psychiatric outcomes. H4: Model performance for predicting which youth move from low to high risk one year in the future will not decrease upon deployment. H5: Performance will remain consistent across settings, and usability will be high as determined by adherence rates and provider and participant surveys.

Exploratory Aim 3:
To explore the use of digital envirotyping to enhance Duke-PMA’s predictive capacity while maintaining resource-efficiency. Integrating AI analysis of photographs of participants’ home environments (modeled after the HOME scale1) will enhance the predictive accuracy of Duke-PMA for psychiatric outcomes, while maintaining accessibility, low cost, and ease of implementation across diverse settings.
 
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
 
### Task 1: Next-Quartile Prediction
**Definition:** Predict which p-factor quartile a participant will be in at
the next follow-up event, from any prior quartile.
 
- **Input:** All predictors at events T, T−1, ..., T−k
- **Output:** P-factor quartile at event T+1
- **Metric:** Macro AUROC (4-class, one-vs-rest)
- **Exclusion:** Participants with no valid p-factor at any follow-up event;
  predictors from the 4-year follow-up (no subsequent event available)
- **Hill et al. reference:** AUROC = 0.80 ± 0.01 (questionnaire model),
  0.89 ± 0.01 (CBCL scales model)
 
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
 
---
 
### Task 4: Agnostic
**Definition:** Predict whether a participant will be in Q4 at the next event,
regardless of their current quartile.
 
- **Input:** All predictors at T and prior events
- **Output:** Binary — is participant in Q4 at T+1?
- **Metric:** AUROC (binary)
- **Exclusion:** Participants with no valid p-factor at T+1
- **Hill et al. reference:** AUROC = 0.80 ± 0.01 (questionnaire model),
  0.89 ± 0.01 (CBCL scales model)
 
---
 
## Evaluation Conventions
 
### Standard Splits
Follow Hill et al.: 8:1:1 train/val/test split, stratified by participant 
to prevent data leakage across longitudinal observations.
All learnable preprocessing (z-score scaling, mean imputation) fit on
training set only.
 
### Metrics
- **Primary:** Macro AUROC for Task 1; binary AUROC for Tasks 2-4
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
 
## Organization Structure
 
```
pma-shared/
  tasks/
    definitions.py        # Task filters, exclusion criteria, label construction
    metrics.py            # AUROC, AP, bootstrap CI, subgroup eval
    pfactor.py            # P-factor computation and quartile binning
  [insert_project_n]/
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
 
---
 
## Notes on Conversion and Persistence Exclusions
 
- **Conversion** excludes Q4 participants at time T because by definition
  they cannot convert (they are already high-risk). Evaluating conversion
  on Q4 participants would conflate it with persistence.
- **Persistence** excludes Q1–Q3 participants at time T because by definition
  they cannot persist in Q4 (they were never in Q4). This is not a data
  quality issue — it is a task definition requirement.

 
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
