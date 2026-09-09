# FIFA World Cup 2026 — Statistical Analysis Assignment

Four independent analytic tasks on FIFA World Cup 2026 data, each answering a distinct
question with the full analysis pipeline: **question formulation → data wrangling → sampling
→ descriptive statistics → confidence interval → hypothesis test (t-test)**.

## Contents

| # | Notebook | Focal point | Level | Test type |
|---|----------|-------------|-------|-----------|
| 1 | `HIT140_possesion_of_winning_teams.ipynb` | Ball possession advantage of winning teams | Match | One-sample t-test (one-tailed) |
| 2 | `Task_3.ipynb` | Shots on target: Group Stage vs. Knockout | Team-match | Two-sample t-test (Welch's) |
| 3 | `Task1.ipynb` | Assists per 90 min: Forwards vs. Midfielders | Player | Two-sample t-test (Welch's) |
| 4 | `shot_conversion_analysis.ipynb` | Shot conversion efficiency: Home vs. Away | Match (paired) | One-sample t-test (paired) |


Each task investigates a genuinely different metric (possession, shots on target, assists,
conversion efficiency) so all four satisfy the "distinct focal point" requirement — no two
tasks analyse the same underlying quantity.

---

## Data

| File | Used in | Grain | Notes |
|---|---|---|---|
| `data/world_cup_match_data.csv` | Task 1 | 1 row / match | Home/away goals + possession % |
| `World_Cup_2026_clean.xlsx` (`TeamMatch` sheet) | Task 2 | 1 row / team-match | `Stage`, `SOT` columns |
| `data/world_cup_2026_players.csv` | Task 3 | 1 row / player | Header on row 2; `Pos`, `Ast.1` (assists/90) |
| `data/world_cup_match_data_t4.csv` | Task 4 | 1 row / match | 66 columns, incl. shots on target |

*(Fill in the exact source URL(s) the data was pulled from here, per the assignment's
"provided sites" requirement — e.g. FBref / FootyStats / official FIFA stats page.)*

Task 1 and Task 4 both read from a match-level CSV but under two slightly different
filenames/paths (`world_cup_match_data.csv` vs. `world_cup_match_data_t4.csv`) — worth
confirming these are the same underlying export (104 total matches in Task 4's file) before
submission, and consolidating the `data/` folder structure so every notebook resolves its
relative path correctly (`../../data/...` vs. `../World_Cup_2026_clean.xlsx`).

### Requirements
```
python >= 3.10
pandas, numpy, scipy, matplotlib, seaborn, openpyxl
```

---

## Task 1 — Possession Advantage of Winning Teams

**Question:** Do winning teams hold significantly greater ball possession than the team they beat?

- **Wrangling:** Loaded match CSV, dropped drawn matches (no winner to compare), derived
  `possession_difference` = winner's possession % − loser's possession %.
- **Population:** 80 decisive matches.
- **Sample:** Simple random sample, n = 30.
- **Descriptives:** mean = 12.80, sd = 22.74, median = 15.0, range −58 to 46.
- **95% CI:** (4.31, 21.29).
- **Test:** One-sample, one-tailed t-test vs. μ = 0 → t(29) = 3.083, **p = 0.0022**.
- **Result:** Reject H₀. **Winning teams hold significantly more possession** than the
  teams they beat.

## Task 2 — Shots on Target: Group Stage vs. Knockout Stage

**Question:** Do teams register a significantly different number of shots on target per
match in knockout-stage vs. group-stage matches?

- **Wrangling:** `TeamMatch` sheet, kept `Stage` and `SOT`, dropped missing SOT.
- **Population:** 206 team-match records (144 group stage, 62 knockout).
- **Sample:** Stratified random sample, 25 per stage (n = 50 total).
- **Descriptives:** Group Stage mean = 3.40, sd = 2.10; Knockout mean = 4.24, sd = 2.50.
- **95% CIs:** Group (2.53, 4.27); Knockout (3.21, 5.27) — overlapping.
- **Test:** Welch's two-sample t-test → t = −1.285, **p = 0.2053**.
- **Result:** Fail to reject H₀. Knockout matches show a numerically higher average, but
  the difference **is not statistically significant** — plausibly sampling variation.

## Task 3 — Assists per 90 Minutes: Forwards vs. Midfielders

**Question:** Do forwards and midfielders have significantly different average assists
per 90 minutes?

- **Wrangling:** Loaded player CSV, kept `Player`, `Pos`, `Ast.1` → renamed `Assists_per_90`;
  filtered to FW/MF only.
- **Population:** 244 players (162 MF, 82 FW), no missing values.
- **Sample:** Simple random sample, n = 40 per position.
- **Descriptives:** FW mean = 0.032, sd = 0.115; MF mean = 0.068, sd = 0.198.
- **95% CI for mean difference (FW − MF):** (−0.108, 0.037) — includes zero.
- **Test:** Welch's two-sample t-test → t = −0.988, **p = 0.327**.
- **Result:** Fail to reject H₀. **No significant difference** in assists per 90 between
  forwards and midfielders in this sample.

## Task 4 — Shot Conversion Efficiency: Home vs. Away

**Question:** Is there a significant difference between home and away teams in shot
conversion efficiency (goals per shot on target)?

- **Wrangling:** Trimmed to goals + SOT columns; engineered `home_conv`/`away_conv`/`diff`;
  dropped 9 matches with zero SOT on either side (undefined ratio) — checked for own-goal
  contamination (conversion > 1: none found).
- **Population:** 95 usable matches (of 104 total).
- **Sample:** Simple random sample, n = 40 (fixed seed).
- **Descriptives:** Home conv. mean = 0.391, sd = 0.321; Away conv. mean = 0.252, sd = 0.247;
  paired diff mean = 0.140, sd = 0.381.
- **Normality check:** Shapiro–Wilk on the paired diff, W = 0.979, p = 0.661 — normality
  not rejected, mean-based test justified.
- **95% CI for the paired diff:** (0.018, 0.262) — excludes zero.
- **Test:** Paired one-sample t-test vs. μ = 0 → t(39) = 2.322, **p = 0.0256**.
- **Robustness check:** Re-run on the full 95-match usable population (no sampling) →
  t = 0.889, p = 0.376 — **not** significant at the population level.
- **Result:** In the 40-match sample, home teams convert shots to goals significantly
  better than away teams (reject H₀). This **doesn't replicate** on the full 95-match
  population, which is flagged in the notebook as a genuine sampling-variability finding
  rather than an error — a useful discussion point on sample size and the risk of a
  single sample overstating an effect.
- **Caveat noted in-notebook:** "home"/"away" reflects fixture designation (shirt colour,
  nominal hosting), not true home-advantage, since 2026 hosts are USA/Canada/Mexico and
  most other teams are technically neutral-venue "away" fixtures.

---

## Results at a Glance

| Task | Metric | Sample n | Mean diff / stat | 95% CI | p-value | Verdict |
|---|---|---|---|---|---|---|
| 1 | Possession diff (winner − loser) | 30 | 12.80 | (4.31, 21.29) | 0.0022 | Significant |
| 2 | SOT: Knockout − Group | 25 + 25 | 0.84 | Overlapping CIs | 0.2053 | Not significant |
| 3 | Assists/90: FW − MF | 40 + 40 | −0.036 | (−0.108, 0.037) | 0.327 | Not significant |
| 4 | Conversion: Home − Away | 40 | 0.140 | (0.018, 0.262) | 0.0256 | Significant (sample); not at population level |

## Methodology Checklist (per task)

Each notebook contains all six required components:

- [x] Analytic question formulation (with explicit H₀/H₁)
- [x] Data wrangling (column selection, missing-value handling, feature engineering)
- [x] Data preparation and sampling (defined population, drawn sample with fixed seed)
- [x] Descriptive statistics (mean, sd, median, quartiles, etc.)
- [x] Inferential statistics — 95% confidence interval
- [x] Inferential statistics — t-test (one-sample or two-sample as appropriate)

## How to Run

1. Place the data files listed above under a shared `data/` folder and update each
   notebook's file path if needed (see the data-consistency note above).
2. `pip install pandas numpy scipy matplotlib seaborn openpyxl`
3. Run notebooks top-to-bottom; all samples use fixed `random_state`/`seed` values for
   reproducibility (1, 7, 42 depending on the notebook).

## Limitations / Discussion Points

- **Sample size trade-off (Task 4):** the clearest illustration in this set of how a
  modest sample can produce a "significant" result that a larger pull of the same
  population does not replicate — worth highlighting explicitly in the write-up as a
  lesson on statistical power rather than downplaying it.
- **"Home" advantage definition (Task 4):** fixture designation ≠ true home advantage for
  most teams at a tri-host tournament; the notebook flags this explicitly.
- **File/task numbering mismatch:** see the note under Contents above — align before
  final submission.
- **Independence assumption (Task 2):** stratified sampling by stage guarantees balanced
  group sizes but relies on team-match SOT records being independent draws; teams appearing
  in multiple matches introduce some within-team correlation worth acknowledging.