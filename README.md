# Who Delegates to AI? — Replication Data and Code

Replication material for *Who Delegates to AI? Evidence from Agent Configurations in
GitHub*. The repository contains one occupation-level dataset and a single notebook that
reproduces the tables and figures.

- `automation_data.csv` — the merged, occupation-level dataset (774 rows, one per SOC
  occupation).
- `automation.ipynb` — loads the dataset and produces the descriptive table, the
  regressions, and the figures.

The core measure is the **Agentic Adoption Index (AAI)**: for each occupation it records
how closely the occupation's O\*NET task statements match the descriptions of
practitioner-built agent skill files shared on public GitHub, aggregated with O\*NET task
importance weights. Higher AAI means an occupation's tasks are, on average, well covered
by agentic skills that early-adopting practitioners have already built and published. The
index is meant for comparison **across** occupations, not for interpretation on an
absolute scale.

## Getting started

```bash
pip install pandas numpy matplotlib seaborn adjustText scipy statsmodels plotly
jupyter notebook automation.ipynb
```

The notebook currently reads the dataset from an absolute path in the first data cell.
Change that one line to point at the shipped file:

```python
merged_df = pd.read_csv('automation_data.csv')
```

The U.S. state-level supplementary section additionally reads a BLS state employment file
(`state_M2025_dl.xlsx`) that is not redistributed here; download it from the BLS OEWS site
if you want to reproduce that section. Everything else runs from `automation_data.csv`
alone.

## Dataset: `automation_data.csv`

One row per occupation, keyed on `soc_code` (774 occupations). Columns fall into five
groups: identifiers, the AAI, the comparison exposure measures, labor-market covariates,
and an O\*NET ability summary. Coverage varies across columns because each source maps to
occupations slightly differently, so some columns have missing values.

### Identifiers

| Column | Description |
|---|---|
| `soc_code` | Base O\*NET-SOC occupation code (e.g. `11-1011`). Merge key; unique per row. |
| `title` | Occupation title, title case (from the AAI/O\*NET source). |
| `occupation` | Occupation name, sentence case (from the Frey–Osborne source where available, otherwise filled from `title`). Used for figure labels. |

### Agentic Adoption Index (this paper's measure)

| Column | Description |
|---|---|
| `aai` | **Agentic Adoption Index.** Importance-weighted mean semantic similarity between the occupation's O\*NET tasks and ~888k GitHub agent skill descriptions, embedded with Sentence-BERT (`all-MiniLM-L6-v2`). Main measure throughout. |
| `aai_manus` | Alternative AAI computed on a different skill corpus/weighting (robustness variant). Not used in the main tables; provided for comparison. |

### Comparison exposure measures (external sources)

These situate the AAI against the three "layers" of AI exposure — capability, availability,
and observed use — plus the pre-AI computerization baseline.

| Column | Layer / source | Description |
|---|---|---|
| `frey_probability` | Pre-AI baseline — Frey & Osborne (2017) | Probability that an occupation is computerizable, estimated before LLMs. |
| `rank` | Frey & Osborne (2017) | Occupation's rank in the Frey–Osborne appendix (by `frey_probability`). Bookkeeping only. |
| `label` | Frey & Osborne (2017) | Sparse hand-label from the appendix; **not** used in analysis. |
| `dv_rating_gamma` | Availability — Eloundou et al. (2024) | GPT-4-rated share of an occupation's tasks for which an LLM **plus complementary software** could cut task time by ≥50% (the γ exposure rubric). Main availability measure. |
| `dv_rating_alpha` | Availability — Eloundou et al. (2024) | Same rubric, α variant: LLM **alone**, no complementary software. Used in the SI. |
| `dv_rating_beta` | Availability — Eloundou et al. (2024) | β variant of the same rubric. Provided for completeness; not used in the main text. |
| `human_rating_alpha` / `human_rating_beta` / `human_rating_gamma` | Availability — Eloundou et al. (2024) | Human-annotated (rather than model-rated) counterparts of the α/β/γ exposure scores. Provided for completeness. |
| `rl_feasibility` | Capability — Tomei & Klein Teeselink (2026) | RL Feasibility Index: suitability of an occupation's tasks for reinforcement-learning-based automation, scored with Gemini 2.5 Flash. Note the large scale relative to the others. |
| `claude_usage` | Observed use — Massenkoff & McCrory (2026) | Occupation-level intensity of actual Claude usage. Concentrated near zero with a long right tail. |

### Labor-market covariates (BLS OEWS)

| Column | Description |
|---|---|
| `employment_2025` | Total employment, 2025 (persons). |
| `median_wage_2025` | Annual median wage, 2025 (USD). Log of this is the wage regressor. |
| `employment_2010` | Total employment, 2010 (persons). Used in the employment-distribution figure. |
| `median_wage_2010` | Annual median wage, 2010 (USD). |
| `education_level` | Typical entry-level education (BLS categories, e.g. "Bachelor's degree"). The notebook groups these into three tiers: high school or below, bachelor's, and master's or above. |

### O\*NET ability summary

| Column | Description |
|---|---|
| `cognitive_ability_share` | Share of an occupation's total O\*NET ability importance that comes from cognitive (vs. physical/psychomotor) abilities. Used to validate that high-AAI occupations are cognitively oriented. |

## What the notebook produces

1. **Descriptive statistics** for the AAI and the three comparison measures.
2. **AAI vs. pre-AI computerization risk** — a Jaccard overlap statistic and the quadrant
   scatter (`aai` vs. `frey_probability`).
3. **Employment distribution** across the Frey–Osborne probability (2010 employment) and
   the AAI (2025 employment).
4. **AAI vs. the three exposure measures** — Spearman correlations, plus the
   cognitive-ability-share validation.
5. **Employment-weighted WLS regressions** of the AAI on wages, education, and availability
   exposure, with a wage × education interaction, and an SI table repeating the
   specifications with the alternative exposure measures.
6. **U.S. state-level supplement** (requires the external BLS state file noted above).

Regressions are weighted by 2025 employment, so estimates describe patterns across the
workforce rather than across job titles. Sample sizes differ across models because each
model drops rows missing any of its own variables.

## Sources

- National Center for O\*NET Development. *O\*NET 30.2 database*, 2025. https://www.onetcenter.org
- Destefanis, G., Graziotin, D., Vaccargiu, M., & Ortu, M. (2026). *GitSkills: A dataset of agent skills on GitHub.* arXiv:2608.10906. https://arxiv.org/abs/2608.10906
- Frey, C. B., & Osborne, M. A. (2017). The future of employment: How susceptible are jobs to computerisation? *Technological Forecasting and Social Change*, 114, 254–280.
- Eloundou, T., Manning, S., Mishkin, P., & Rock, D. (2024). GPTs are GPTs: Labor market impact potential of LLMs. *Science*, 384(6702), 1306–1308. doi:10.1126/science.adj0998
- Tomei, P. M., & Klein Teeselink, B. (2026). *What jobs can AI learn? Measuring exposure by reinforcement learning.* arXiv:2605.02598.
- Massenkoff, M., & McCrory, P. (2026). *Labor market impacts of AI: A new measure and early evidence.* https://www.anthropic.com/research/labor-market-impacts
- Bureau of Labor Statistics. *Occupational Employment and Wage Statistics (OEWS)*, May 2010 and May 2025. https://www.bls.gov/oes/

Each external exposure measure is redistributed here only as merged into the
occupation-level table; refer to the original sources for full documentation and terms of
use.
