# llm-ahp-mcdm-study: LLM respondents in AHP supplier selection (data, results and reports)

This repository holds the complete records, input files and generated reports of one study: nine large
language models answered the Analytic Hierarchy Process (AHP) pairwise-comparison questions of a
supplier-selection problem (5 main criteria, 4 delivery sub-criteria) under three prompt strategies, and their
weights were compared with those of seven human experts and used to rank five suppliers with TOPSIS, ARAS
and EDAS. The code that produced and checks every number is in
[llm-ahp-mcdm](https://github.com/baa-git/llm-ahp-mcdm) (version v1.0.0); this repository contains no code.

- **Models (OpenRouter):** claude-haiku-4.5, claude-sonnet-4.6, deepseek-v3.2, gemini-3.1-flash-lite, glm-5,
  gpt-4o, gpt-5.2, minimax-m2.5, qwen3.5-397b; temperature 0.6; judge model x-ai/grok-4.20 for revisions.
- **Conditions:** Guided (zero-shot), Unguided, zero-shot (direct) and Unguided, CoT (see below).
- **Design:** 9 models x 3 conditions x 2 tasks x 5 repetitions = 270 first-answer calls (run tag `main`);
  direct repeated at temperature 0.0 and 1.0 (3 repetitions each); inconsistent matrices (CR > 0.10) of the
  direct and CoT conditions revised with code-computed feedback phrased by a judge model (at most 5
  revisions, run tag `revision_final`).
- **Reference:** seven experts' AHP matrices (E1-E7, anonymised).

## Conditions

Conditions are described on two axes: prompting technique (zero-shot = no worked examples and no reasoning instruction; chain-of-thought = an instruction to reason step by step) and weight guidance (guided = the prompt states a priority order and importance hints; unguided = neutral questions). Short labels: guided, direct (stored key `lean`) and CoT (stored key `cot`).

| Technique / guidance | Guided | Unguided |
| --- | --- | --- |
| Zero-shot | Guided (zero-shot): `guided` | Unguided, zero-shot (direct): `lean` |
| Chain-of-thought | not run | Unguided, CoT: `cot` |

| Prompt content | Guided (zero-shot) | Unguided, zero-shot (direct) | Unguided, CoT |
| --- | --- | --- | --- |
| Role line | AHP expert specialised in supplier evaluation, with AHP rules | procurement professional with experience in supplier evaluation | same as direct |
| Priority order | yes: typical priority order (Quality highest ... Volume lower); "never use all 1s" | no | no |
| Criterion hints | yes: importance-laden descriptions of the criteria (in Turkish) | no (names only) | no (names only) |
| KPI report | yes: data report on the suppliers (in Turkish) | no | no |
| Reasoning instruction | rationale per comparison requested | none | reason step by step and state a ranking before answering |
| Reply format | full pairwise matrix with labels and rationales (JSON) | one value per question (JSON) | reasoning, ranking and one value per question (JSON) |

## Results at a glance

All numbers below are copied from the CSV tables in `results/report/tables/` (Txx); `results/report/report.md`
has every table with its full description. Agreement with the expert weights is a similarity measure, not a
quality measure: a condition closer to the expert weights is not better for that reason.

### Consistency (T03, T02)

Number of matrices with CR ≤ 0.10 (CR rounded to 4 decimals), mean CR, and CR of the geometric-mean group
matrix, first answers.

| Respondents | CR ≤ 0.10 main | Mean CR main | Group CR main | CR ≤ 0.10 delivery | Mean CR delivery | Group CR delivery |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| LLM panel, Guided (zero-shot) | 43/45 | 0.0435 | 0.0315 | 45/45 | 0.0252 | 0.0033 |
| LLM panel, Unguided, zero-shot (direct) | 15/45 | 0.2115 | 0.0616 | 23/45 | 0.2968 | 0.0866 |
| LLM panel, Unguided, CoT | 24/45 | 0.2101 | 0.0228 | 29/45 | 0.0968 | 0.0075 |
| Experts (7) | 4/7 |  | 0.0321 | 0/7 |  | 0.0208 |

### Group weights (T04, T06)

Main-criteria weights (geometric-mean group matrix of all 45 matrices per condition; experts: 7 matrices):

| Main criterion | Experts | Guided (zero-shot) | Unguided, zero-shot (direct) | Unguided, CoT |
| --- | ---: | ---: | ---: | ---: |
| Delivery and Operational Performance | 0.2147 | 0.1555 | 0.1744 | 0.1864 |
| Financial Performance | 0.1604 | 0.0721 | 0.0796 | 0.0950 |
| Quality Performance | 0.3186 | 0.4651 | 0.3507 | 0.3757 |
| Volume and Logistics | 0.0529 | 0.0451 | 0.0870 | 0.0800 |
| Compliance and Reliability | 0.2535 | 0.2622 | 0.3083 | 0.2629 |

Global weights (delivery local weights times the delivery main weight; the other four main weights unchanged):

| Global weight | Experts | Guided (zero-shot) | Unguided, zero-shot (direct) | Unguided, CoT |
| --- | ---: | ---: | ---: | ---: |
| D1 Delivered | 0.0539 | 0.0957 | 0.0853 | 0.1038 |
| D2 Pending | 0.0745 | 0.0220 | 0.0232 | 0.0232 |
| D3 Cancelled | 0.0550 | 0.0156 | 0.0302 | 0.0281 |
| D4 Partially delivered | 0.0313 | 0.0222 | 0.0358 | 0.0313 |
| F Financial | 0.1604 | 0.0721 | 0.0796 | 0.0950 |
| Q Quality | 0.3186 | 0.4651 | 0.3507 | 0.3757 |
| V Volume | 0.0529 | 0.0451 | 0.0870 | 0.0800 |
| C Compliance | 0.2535 | 0.2622 | 0.3083 | 0.2629 |

### Similarity to the expert weights (T08)

RMSE and mean absolute difference (MAD) in percentage points, Spearman rho and Kendall tau of the panel
weights against the expert weights, for the five main and the eight global weights.

| Condition | Weights | RMSE (pp) | MAD (pp) | Spearman rho | Kendall tau |
| --- | ---: | ---: | ---: | ---: | ---: |
| Guided (zero-shot) | main | 8.11 | 6.21 | 1.000 | 1.000 |
| Unguided, zero-shot (direct) | main | 5.17 | 4.84 | 0.900 | 0.800 |
| Unguided, CoT | main | 4.28 | 3.75 | 1.000 | 1.000 |
| Guided (zero-shot) | global | 6.67 | 4.93 | 0.571 | 0.500 |
| Unguided, zero-shot (direct) | global | 4.47 | 3.92 | 0.429 | 0.286 |
| Unguided, CoT | global | 4.22 | 3.59 | 0.548 | 0.429 |

### Supplier rankings (T11, T13, T21)

Rankings of the five suppliers with the delivery indicators as shares of each supplier's orders, under the
expert, panel and equal global weights, and the agreement of each ranking with the expert-weight ranking
(Spearman rho, Kendall tau, number of suppliers whose rank differs).

| Method | Weights | Ranking (rank 1 = best ... last = worst) | Spearman rho vs experts | Kendall tau vs experts | Suppliers with a different rank |
| --- | ---: | ---: | ---: | ---: | ---: |
| TOPSIS | Experts | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | reference |  |  |
| TOPSIS | LLM panel, Guided (zero-shot) | Alpha_Inc > Epsilon_Group > Gamma_Co > Beta_Supplies > Delta_Logistics | 0.900 | 0.800 | 2 |
| TOPSIS | LLM panel, Unguided, zero-shot (direct) | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| TOPSIS | LLM panel, Unguided, CoT | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| TOPSIS | Equal (1/8) | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| ARAS | Experts | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | reference |  |  |
| ARAS | LLM panel, Guided (zero-shot) | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| ARAS | LLM panel, Unguided, zero-shot (direct) | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| ARAS | LLM panel, Unguided, CoT | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| ARAS | Equal (1/8) | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| EDAS | Experts | Alpha_Inc > Epsilon_Group > Gamma_Co > Beta_Supplies > Delta_Logistics | reference |  |  |
| EDAS | LLM panel, Guided (zero-shot) | Alpha_Inc > Epsilon_Group > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |
| EDAS | LLM panel, Unguided, zero-shot (direct) | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 0.900 | 0.800 | 2 |
| EDAS | LLM panel, Unguided, CoT | Epsilon_Group > Alpha_Inc > Gamma_Co > Beta_Supplies > Delta_Logistics | 0.900 | 0.800 | 2 |
| EDAS | Equal (1/8) | Alpha_Inc > Epsilon_Group > Gamma_Co > Beta_Supplies > Delta_Logistics | 1.000 | 1.000 | 0 |

### Revision outcomes (T16) and logic preservation (T20)

Panel rows of the revision run (`revision_final`); "final" means the best valid answers of a chain (lowest
CR among the first answers and all parsable revisions).

| Condition | Task | Needing revision (CR > 0.10) | Reached CR ≤ 0.10 after 1 | after ≤ 5 | Not reached | Mean initial CR | Mean final CR |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Unguided, zero-shot (direct) | main | 30 | 8 | 19 | 11 | 0.2884 | 0.0971 |
| Unguided, zero-shot (direct) | delivery | 22 | 5 | 17 | 5 | 0.5356 | 0.0907 |
| Unguided, CoT | main | 21 | 7 | 17 | 4 | 0.3820 | 0.0751 |
| Unguided, CoT | delivery | 16 | 11 | 16 | 0 | 0.1726 | 0.0443 |

Comparison of the final with the first answers of each revised chain, and the judge's verdicts on the
revision that gave the final answers:

| Condition | Task | Chains revised | Answers changed (mean) | Spearman rho first vs final | Top criterion unchanged | Full ranking unchanged | Largest weight change (pp) | Judge yes / partly / no |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Unguided, zero-shot (direct) | main | 30 | 4.233 | 0.813 | 0.8000 | 0.4000 | 9.06 | 7 / 13 / 8 |
| Unguided, zero-shot (direct) | delivery | 22 | 3.955 | 0.626 | 0.6364 | 0.1818 | 16.46 | 1 / 6 / 15 |
| Unguided, CoT | main | 21 | 5.333 | 0.714 | 0.6190 | 0.1905 | 15.86 | 7 / 3 / 11 |
| Unguided, CoT | delivery | 16 | 4.250 | 0.675 | 0.9375 | 0.3750 | 10.91 | 8 / 2 / 6 |

### Replies and the order stated in the guided prompt (T22)

The guided main-criteria prompt states a typical priority order (Quality > Compliance > Delivery > Financial >
Volume, read from `instructions.priority_guidance` of the prompt file); the guided delivery prompt names
Delivered Orders as the primary positive indicator (no full order). The table counts the replies that followed
the prompted order. Rows marked "reference" never saw this order.

| Task | Replies of | Comparison | Replies | Ranking equals prompted order | Prompted first criterion ranked first | Mean Spearman rho |
| --- | ---: | ---: | ---: | ---: | ---: | ---: |
| main | LLM panel, Guided (zero-shot) | prompted | 45 | 40 | 42 | 0.987 |
| main | LLM panel, Unguided, zero-shot (direct) | reference (order not shown) | 45 | 6 | 21 | 0.776 |
| main | LLM panel, Unguided, CoT | reference (order not shown) | 45 | 7 | 24 | 0.711 |
| main | Experts (individual matrices) | reference (order not shown) | 7 | 1 | 2 | 0.671 |
| delivery | LLM panel, Guided (zero-shot) | prompted | 45 | - | 45 | - |
| delivery | LLM panel, Unguided, zero-shot (direct) | reference (order not shown) | 45 | - | 27 | - |
| delivery | LLM panel, Unguided, CoT | reference (order not shown) | 45 | - | 41 | - |
| delivery | Experts (individual matrices) | reference (order not shown) | 7 | - | 3 | - |

## Where each file is

```
results/raw/            the study's records (JSONL, one object per call; see the data dictionary)
  calls.jsonl             first-answer calls: main, temp_0.0, temp_1.0, pilot
  revisions.jsonl         revision calls of the panel models
  judge_calls.jsonl       judge calls (feedback and verdict)
  param_tests.jsonl       parameter probes (gpt-5.2 temperature, minimax-m2.5 reasoning)
  calls_backup_before_reparse.jsonl, calls.jsonl.bak   earlier copies of calls.jsonl (re-parse)
results/*.log           console logs of the runs
results/report/         generated: report.md, tables/ (T01-T22, CSV), appendix/ (A01-A08, CSV, with README.md),
                        figures/ (F1, F2), audit/ (audit.md and CSV), TEKNIK_RAPOR.md (Turkish technical report)
data/                   expert_matrices.json (E1-E7), performance_matrix.json (5 suppliers x 8 indicators)
problems/               supplier_selection.json: the problem definition used by the code
prompts/                guided/ (the original guided prompt files), PROMPT_TASLAK.md (prompt drafts)
models.json             model panel and judge settings of the study
DATA_PROVENANCE.md      sources, run dates and open points
```

`results/report/appendix/README.md` lists every table and appendix file with its content and row count.

## Run tags: results and pilots

| Run tag | File | Records | Status |
| --- | --- | ---: | --- |
| `main` | calls.jsonl | 270 | result (T01-T18, T20-T22) |
| `temp_0.0`, `temp_1.0` | calls.jsonl | 54 + 54 | result (T18) |
| `revision_final` | revisions.jsonl / judge_calls.jsonl | 250 / 500 | result (T16, T17, T20) |
| `pilot` | calls.jsonl | 10 | pilot, not used |
| `revision_judge`, `revision_v2`, `rev_final`, `rev_test_none` | revisions.jsonl / judge_calls.jsonl | 260 / 488 | revision pilots, not used |
| (none) | param_tests.jsonl | 14 | settings probes, cited in TEKNIK_RAPOR and audit k |

## Data dictionary (JSONL records)

Records use the key `lean` for the direct condition (unguided, zero-shot) and `cot` for CoT (unguided,
chain-of-thought), also in prompt ids such as `lean_main_v2` and in the console logs; the generated tables show
the short labels guided / direct / CoT with the columns guidance and technique.

`results/raw/calls.jsonl` - one object per first-answer call:

| Field | Content |
| --- | --- |
| `run_tag` | run label (`main`, `temp_0.0`, `temp_1.0`, `pilot`) |
| `model_key`, `model_id`, `developer` | panel key, OpenRouter model id, developer |
| `condition`, `task`, `rep` | `lean` (direct) / `cot` (CoT) / `guided`; `main` or `delivery`; repetition 1..n |
| `prompt_id`, `prompt_sha256`, `messages` | prompt identifier, SHA-256 of the JSON-serialised messages, the messages sent |
| `criteria`, `temperature_requested` | criteria of the task in order; requested temperature |
| `ok`, `error`, `content`, `raw_reply` | call success, error text, reply text (`raw_reply` = `content`) |
| `model_reasoning_text`, `finish_reason`, `native_finish_reason` | reasoning text if returned; finish reasons |
| `usage` | `prompt_tokens`, `completion_tokens`, `reasoning_tokens`, `total_tokens` |
| `cost`, `provider`, `generation_id` | billed USD (OpenRouter `usage.cost`), serving provider, generation id |
| `requested_params`, `applied_params` | temperature / reasoning / max_tokens requested and finally applied (`omitted` lists dropped parameters) |
| `http_attempts` | every HTTP attempt: status, parameters, seconds, error |
| `latency_s`, `timestamp_utc` | total latency; start time (UTC, ISO 8601) |
| `parse_ok`, `parse_error`, `bare_fractions_quoted`, `parser_version` | parsing result and flags |
| `answers`, `reasoning`, `ranking` | the answers per question (17-level scale); CoT reasoning and stated ranking |
| `matrix`, `weights`, `lambda_max`, `CI`, `CR` | reciprocal matrix, eigenvector weights (criterion -> weight), consistency |
| `labels`, `label_order`, `labels_verified`, `reordered`, `raw_matrix`, `rationales`, `reciprocal_returned`, `upper_on_scale`, `guided_labels_expected` | guided condition only: the returned matrix and how it was mapped |

`results/raw/revisions.jsonl` - one object per revision call: the fields above plus `source_tag`,
`revision_number`, `max_revisions`, `source_CR`, `based_on_answers`, `based_on_CR`, `based_on_revision`,
`best_CR_so_far` and `judge_model_id`, `judge_feedback`, `judge_feedback_ok`, `judge_feedback_reused`,
`judge_verdict` (`logic_preserved`, `changed_priorities`, `comment`), `judge_verdict_ok`, `judge_cost`,
`logic_metrics`.

`results/raw/judge_calls.jsonl` - one object per judge call: `run_tag`, `source_tag`, `chain_key`,
`chain_model_key`, `condition`, `task`, `rep`, `revision_number`, `purpose` (`feedback` / `verdict`),
`messages`, the client fields, `raw_reply`, `judge_parsed`, `judge_parse_ok`.

`results/raw/param_tests.jsonl` - parameter probes: `test`, `model_key`, `extra` (parameters sent), `status`,
`content`, `error`, `reasoning_tokens`, `cost`.

## How to reproduce

No API calls and no key are needed; everything is recomputed from `results/raw/`. Python 3.10+ with numpy
(matplotlib for the figures).

```
git clone https://github.com/baa-git/llm-ahp-mcdm-study
git clone --branch v1.0.0 https://github.com/baa-git/llm-ahp-mcdm
cd llm-ahp-mcdm
python -m pip install numpy matplotlib
python -m unittest discover -s tests            # offline unit tests (synthetic data)
python -m ahp_bench audit --results ../llm-ahp-mcdm-study/results --out ../check
python -m ahp_bench report --results ../llm-ahp-mcdm-study/results --data ../llm-ahp-mcdm-study/data \
    --tag main --revision-tag revision_final --out ../check
python -m ahp_bench tech-report --results ../llm-ahp-mcdm-study/results --data ../llm-ahp-mcdm-study/data \
    --report ../check
```

The audit writes `../check/audit/audit.md`: 86 expectation checks, all passing (every main record stores
exactly the messages the code builds, weights and CR recompute to within 1e-9, re-parsing the raw replies
gives the same results, no key material is stored, all 89 revision chains follow the protocol). The report
writes the tables, appendix and figures to `../check/`; the CSV tables and appendix files are byte-identical to
those in `results/report/` (compare with `diff -r ../check/tables results/report/tables` from this repository;
figure files can differ byte-wise with another matplotlib version). Omit `--out` / `--report` to regenerate the
files in place. With the code installed (`pip install git+https://github.com/baa-git/llm-ahp-mcdm@v1.0.0`)
the same commands are available as `ahp-bench audit ...`, `ahp-bench report ...`.

## Costs

The whole study cost 4.2660 USD (OpenRouter `usage.cost`): main run 1.4783 USD for 270 calls,
temperature arms 0.0845 and 0.0948 USD, final revision 0.6633 USD (panel) + 0.4735 USD (judge),
the rest pilots and parameter tests (`results/report/TEKNIK_RAPOR.md`, section 9).

## Limitations

- LLM answers are not expert judgement; the panel weights describe these models, prompts and dates (November 2026).
- Five repetitions per cell (three in the temperature arms); model-level statistics rest on few draws.
- Model versions behind an OpenRouter id, provider routing and prices can change; providers were not pinned.
  gpt-5.2 accepted but ignored temperature; minimax-m2.5 could not disable reasoning.
- One problem (supplier selection), one performance data set, seven experts; the questions were asked in
  English to the models and in Turkish to the experts.
- Revision changes priorities; revised weights are a different measurement, not a correction. The judge's
  logic-preservation verdicts come from one model and were not compared with human ratings.
- The guided condition was not revised, and chain-of-thought was not combined with the guided prompt (CoT x guided was not run).

## License and citation

Data and results: CC BY 4.0 (`LICENSE`), with the third-party notes on the Kaggle-derived performance data
and on model outputs given there; sources in `DATA_PROVENANCE.md`. Cite with `CITATION.cff`. Code:
[llm-ahp-mcdm](https://github.com/baa-git/llm-ahp-mcdm) (MIT).
