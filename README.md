# Augmenting Lecture Information via Redundancy-Filtered LLM Slide Descriptions

Data and results accompanying the paper ([INLG 2026 accepted papers](https://2026.inlgmeeting.org/accepted-papers.html)). Lecture slide descriptions are generated
by an LLM and then **redundancy-filtered** against the spoken transcript, so that
the description retains only what a listener cannot get from the audio alone
(diagrams, chart values, table numbers, formulas). The resulting descriptions are
evaluated by how well a question-answering model can answer questions about the
lecture when given transcript + description as context.

This repository contains the **custom QA dataset**, the **generated slide
descriptions**, and the **evaluation results reported in the paper**. Naming
follows the condition labels used in the paper's figures and tables.

## Conditions

Every condition supplies a different context to the QA model. `Transcript-only`
is the baseline with no slide description at all.

| Label | Description generator | Redundancy-filtered | Prompt |
|---|---|---|---|
| `Transcript-only` | — (transcript only, no slide description) | — | — |
| `Gemma-Full-P1` | Gemma 3 12B IT | no | v1 |
| `Gemma-Eff-P1` | Gemma 3 12B IT | yes | v1 |
| `Gemma-Full-P2` | Gemma 3 12B IT | no | v2 |
| `Gemma-Eff-P2` | Gemma 3 12B IT | yes | v2 |
| `Mistral-Eff-v1` | Gemma 3 12B IT → filtered by `mistral-small` | yes | v1 |
| `Mistral-Eff-v2` | Gemma 3 12B IT → filtered by `mistral-small` | yes | v2 |
| `Mistral-Eff-v3` | Gemma 3 12B IT → filtered by `mistral-small` | yes | v3 |

`Gemma-*` conditions filter for redundancy inside the description prompt itself.
`Mistral-Eff-*` conditions start from the unfiltered `Gemma-Full-P2` descriptions
and remove transcript-redundant content in a second `mistral-small` pass; v1/v2/v3
are three variants of that filtering prompt.

## Layout

```
custom_dataset/
  custom.src.xml                  53 questions (the <instruction> elements)
  custom.ref.xml                  53 reference answers, with qa_type per item
slide_descriptions_generated/
  transcripts/                    18 lecture transcripts (.en)
  slide_descriptions_*/           the generated descriptions, 18 files each
results/
  qa_answers/<condition>.xml      QA model answers per condition (the hypothesis files)
  llm_judge_mistral-small/        primary judge: per-item scores + summaries
  llm_judge_gpt-5.4/              second judge, for interrater reliability
  llm_judge_ministral-14b/        third judge, for interrater reliability
  interrater_reliability/         judge-vs-judge and human-vs-judge agreement
  human_rating/                   the human rating subsample and the human ratings
  description_efficiency/         word counts, uniqueness, compression ratios
  statistical_tests_llm_judge.csv                 pairwise tests on judge scores
  statistical_tests_description_uniqueness.csv    pairwise tests on uniqueness
figures/                          the figures used in the paper
```

### Description directory → condition label

The description directories keep their generation-time names; this is the mapping
to the paper labels:

| Directory under `slide_descriptions_generated/` | Label |
|---|---|
| `slide_descriptions_non-efficient_gemma3_desc_prompt1` | `Gemma-Full-P1` |
| `slide_descriptions_efficient_gemma3_des_prompt1` | `Gemma-Eff-P1` |
| `slide_descriptions_non-efficient_gemma3_desc_prompt2` | `Gemma-Full-P2` |
| `slide_descriptions_efficient_gemma3_desc_prompt2` | `Gemma-Eff-P2` |
| `slide_descriptions_efficient_with_mistral_prompt_v1` | `Mistral-Eff-v1` |
| `slide_descriptions_efficient_with_mistral_prompt_v2` | `Mistral-Eff-v2` |
| `slide_descriptions_efficient_with_mistral_prompt_v3` | `Mistral-Eff-v3` |

Description and transcript files are named by lecture id (e.g. `BIdHavzwsU`),
which matches the `<text_path>` element in `custom.ref.xml`.

## File formats

**`results/qa_answers/<condition>.xml`** — one `<sample id="…">` per question,
containing the QA model's answer. Ids align with `custom.src.xml` /
`custom.ref.xml`.

**`results/llm_judge_*/scores_<condition>.csv`** — one row per question:

| column | meaning |
|---|---|
| `sample_id`, `iid` | question id (aligns with the dataset XMLs) |
| `qa_type` | `text-extraction`, `graph-related`, `table-extraction`, `formula-extraction` |
| `reference`, `hypothesis` | reference answer and the model answer being scored |
| `llm_score`, `llm_reason` | judge score in {0.0, 0.5, 1.0} and its justification |
| `bertscore_f1`, `rouge_l` | reference-based similarity metrics |
| `faithfulness_score`, `faithfulness_reason` | whether the answer is grounded in the given context |

The `gpt-5.4` and `ministral-14b` directories carry `llm_score` only — they exist
to establish interrater reliability, and the similarity metrics do not depend on
the judge.

**`results/interrater_reliability/*.csv`** — per condition plus a pooled
`OVERALL` row: `n`, `pct_agree`, `kappa_linear` (linear-weighted Cohen's κ),
`spearman_rho`, `spearman_p`. The human-vs-judge files cover the 80-item rated
subsample; the judge-vs-judge files cover the full evaluation set.

## Results

Average scores under the primary judge (`mistral-small`), 53 questions per condition:

| Condition | LLM judge | BERTScore F1 | ROUGE-L | Faithfulness |
|---|---|---|---|---|
| `Transcript-only` | 0.302 | 0.830 | 0.137 | 0.566 |
| `Gemma-Eff-P1` | 0.434 | 0.836 | 0.180 | 0.528 |
| `Gemma-Full-P1` | 0.462 | 0.841 | 0.214 | 0.623 |
| `Gemma-Eff-P2` | 0.509 | 0.843 | 0.235 | 0.547 |
| `Gemma-Full-P2` | 0.547 | 0.837 | 0.206 | 0.519 |
| `Mistral-Eff-v1` | 0.528 | 0.846 | 0.235 | 0.500 |
| `Mistral-Eff-v2` | 0.415 | 0.844 | 0.204 | 0.557 |
| `Mistral-Eff-v3` | **0.566** | 0.846 | 0.232 | 0.566 |


### Judge reliability

Pooled linear-weighted Cohen's κ and Spearman ρ:

| Pair | n | κ | ρ |
|---|---|---|---|
| `mistral-small` vs `ministral-14b` (same family) | 419 | 0.778 | 0.848 |
| `mistral-small` vs `gpt-5.4` | 419 | 0.772 | 0.850 |
| human vs `gpt-5.4` | 80 | 0.764 | 0.834 |
| human vs `mistral-small` | 80 | 0.698 | 0.798 |

The two `n=419` rows exclude 5 items whose stored QA answer is an API-error
string rather than a real answer. 

### Human rating

`results/human_rating/` holds a blinded subsample of 80 items (10 per condition),
stratified by condition and question type with a fixed random seed. Raters saw
neither the condition nor the automatic scores; `rating_key.csv` maps each
`item_id` back to its condition and the automatic scores.

- `ratings_human_first-author.csv` — all 80 items, rated by the first author

## Other sources

The lecture material is from the
[MCIF benchmark](https://huggingface.co/datasets/FBK-MT/MCIF) (FBK-MT).
