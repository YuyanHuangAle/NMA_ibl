# Learning and choice history in the IBL mouse decision-making dataset

Analysis of training behaviour in the International Brain Laboratory (IBL) standardised
decision-making task, asking how reward, choice history and sensory evidence shape learning
and trial-by-trial choices.

Work completed during the Neuromatch Academy Computational Neuroscience course (July 2026)
and posted afterwards(September 2026).

---

## Question

Mice in the IBL task learn to report the side of a visual stimulus over weeks of training.
This repository asks three things of the publicly released behavioural data:

1. **How does performance improve across training**, and can the trajectory be summarised by a
   single learning rate per mouse?
2. **Does early reward exposure predict how fast a mouse learns?**
3. **How much does a mouse's previous choice influence its current one**, relative to the
   sensory evidence actually available on that trial?

## Data

Public IBL behavioural data, accessed via the [ONE API](https://int-brain-lab.github.io/ONE/)
against the `openalyx.internationalbrainlab.org` database.

- 140 subjects across [N] labs
- [N] training sessions
- 418,082 trials in the GLM analysis

Data are released openly by the International Brain Laboratory. See **Attribution** below.

## Methods

**Learning curves.** `performance_easy` (proportion correct on high-contrast trials) computed
per session, then fitted per mouse with an exponential of the form `a - b·exp(-k·t)`. The
recovered rate constant `k` is used as a per-mouse learning rate.

**Reward measures.** Two distinct quantities, which behave differently and should not be
conflated:

- *mean reward across all trials* — average reward the mouse actually earned across trials
- *mean reward when rewarded* — the reward volume per correct trial (consistent for one session)

**Choice-history GLM.** Binomial GLM (logit link) predicting rightward choice from:

- `signed_contrast_right` — current sensory evidence
- `previous_choice_signed` — choice persistence
- `win_stay_lose_switch` — reward-dependent updating

Standard errors clustered by [subject / session] to account for repeated measures.

**Statistics.** Pearson and Spearman correlations reported together throughout, since several
relationships are monotonic but not linear. Paired Wilcoxon signed-rank test for the
win-stay / lose-stay comparison.

## Key results

| Finding                                                | Statistic                       |
| ------------------------------------------------------ | ------------------------------- |
| Early reward exposure vs early performance improvement | Pearson r = 0.634, p = 1.9e-13  |
| Early reward exposure vs full learning rate `k`        | Pearson r = 0.651, p = 2.3e-14  |
| Early reward **size** vs early performance improvement | Pearson r = −0.318, p = 8.1e-04 |
| Previous-choice persistence (GLM coefficient)          | 0.276, z = 9.34, p < 0.001      |
| Current sensory evidence (GLM coefficient)             | 0.780, z = 14.79, p < 0.001     |
| Reward-dependent updating (GLM coefficient)            | −0.062, z = −2.30, p = 0.022    |
| Choice repetition after reward vs after error          | Paired Wilcoxon p = 1.3e-04     |

Current sensory evidence remains the strongest single predictor of choice, but previous-choice
persistence is substantial and independently significant. Mice tended to repeat their previous
choice after both rewarded and error trials during early training.

Figures: [`results/NMA_analysis.pdf`](results/)

## Limitations

**The reward-exposure correlations are partly confounded.** Total reward earned in early
sessions is itself a consequence of performance: a mouse that gets more trials correct
receives more reward. The relationship between early reward exposure and learning rate is
therefore not evidence that reward *drives* learning, and cumulative reward and trial count
are close to statistically inseparable in this dataset.

The reward-*size* analysis is better identified, because reward volume per correct trial is
set by the experimenter rather than earned by the mouse. That analysis shows a negative
relationship with early improvement.

Reaction time is noisy and poorly described by a simple exponential decay; the fitted
constants for RT should be treated as descriptive only. Log-transforming RT did not improve
this.

## Running the code

```bash
pip install -r requirements.txt
jupyter notebook notebooks/IBL_behavior_data.ipynb
```

The first run downloads aggregate tables for every subject from IBL's S3 store, which takes
[N] minutes. The assembled dataframe is then cached to `data/all_trials_cached.parquet`;
subsequent runs load from cache.

### Known issue: missing aggregate tables

Some subjects are missing one or more of the three aggregate tables
(`_ibl_subjectTrials`, `_ibl_subjectSessions`, `_ibl_subjectTraining`). For example,
`CSHL_001` has a trials table but no sessions table.

The aggregation loop catches these, skips the affected subject and records it. The list of
skipped subjects is printed at the end of the loop and stored in `skipped_subjects.csv`, so
the excluded sample is explicit rather than silent.

ONE API version pinned in `requirements.txt`, since dataset availability on the remote store
changes over time.

## Repository structure

```
notebooks/     IBL_behavior_data.ipynb    main analysis
src/           loading, fitting and plotting functions
results/       figures and presentation PDFs
data/          cached parquet (gitignored)
```

## Attribution

Behavioural data from the International Brain Laboratory. Please cite:

> The International Brain Laboratory et al. (2021). Standardized and reproducible measurement
> of decision-making in mice. *eLife* 10:e63711.

Data portal: https://openalyx.internationalbrainlab.org

Some loading and psychometric-curve code is adapted from IBL's public example notebooks and
from Neuromatch Academy course materials; this is noted inline where it applies.

## Contributions

All code, analyses and figures in this repository are my own.

This work was carried out during a five-person Neuromatch Academy group project on the IBL
decision-making dataset. The wider project also covered [within-session reward-rate effects
on response vigor and arousal / pupillometry analyses], which were led by other team members
and are not included here. This repository contains only my own contributions.

## Author

Yuyan Huang (Alethea) — [link] · MSc Neuroscience, UCL
