# Human rating instructions

You are rating whether a model's answer to a question about a lecture video is
correct, given a reference (ground-truth) answer.

Fill in the **`score`** column of `rating_sheet_raterX.csv` for every row.
Use exactly one of these three values:

| score | meaning |
|-------|---------|
| `1.0` | Correct — conveys the same key information as the reference answer. |
| `0.5` | Partially correct — captures some but not all key information, or has minor errors. |
| `0.0` | Incorrect — wrong information, irrelevant, or misses the point entirely. |

This is the same three-point rubric the automatic judges were given, so please
apply it the same way:

- Judge **content**, not style, length, or fluency. A terse answer that contains
  the key information is `1.0`.
- Judge against the **reference answer**, not against your own knowledge of the
  topic. The `question` column is context to help you interpret the reference —
  it is not the standard.
- If the answer is correct but adds extra material not in the reference, that is
  still `1.0` unless the extra material is wrong or contradicts the reference.
- If you genuinely cannot decide, use the `notes` column to say why, but still
  enter a score. Leave no row blank.

## If the reference answer itself looks wrong

Some reference answers do not actually answer their question (they contain
unrelated content from the talk). Scoring against such a reference would punish
an answer that is in fact correct.

If you hit one, start the `notes` cell with **`REF?`** — for example
`REF? reference is talk content, not a motivation` — and still enter your best
guess in `score`. These items can then be reported both with and without the
disputed references, instead of silently distorting the results.

Use `REF?` only when the reference does not answer the question. If the
reference is merely terse, awkward, or has a typo, rate it normally.

Please do **not** discuss individual items with the other rater before both of
you have finished — the point of the exercise is independent judgment.

The rows are in randomised order and the model/condition that produced each
answer is deliberately hidden from you.

## Columns

- `item_id`      — identifier, do not change
- `qa_type`      — the kind of question (graph-related, text-extraction, …)
- `question`     — the question that was asked
- `reference_answer` — the ground truth
- `model_answer` — the answer to rate
- `score`        — **you fill this in**: 0.0 / 0.5 / 1.0
- `notes`        — optional free text
