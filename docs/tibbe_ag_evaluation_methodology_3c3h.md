# Evaluation Methodology: 3C3H for Tibbe-AG

This document describes how answers in this repository are evaluated. It combines the paper's Evaluation section with the official 3C3H documentation ([AraGen / Hugging Face](https://huggingface.co/blog/leaderboard-3c3h-aragen), El Filali et al., 2024).

## Overview

Every generated answer (across the Direct, RAG, and Tibbe-AG settings, for each of the three base models) is scored by a secondary LLM acting as an **agentic judge**. The judge assigns six per-sample scores that are combined into a single **3C3H** quality score. To confirm robustness, the paper reports scores from four judges: **GPT-4.5** (primary), **O4-mini**, **Claude-4**, and **Gemini**.

## The six dimensions

| # | Dimension | Range | Question it answers |
| --- | --- | --- | --- |
| c1 | **Correctness** | 0 or 1 | Is the answer factually accurate w.r.t. the ground truth? |
| c2 | **Completeness** | 0 or 1 | Does the answer address all parts of the question? |
| c3 | **Conciseness** | 1–5 | Is the answer appropriately brief while retaining necessary detail? |
| h1 | **Helpfulness** | 1–5 | Does the answer effectively assist/inform the user? |
| h2 | **Honesty** | 1–5 | Is all (incl. incidental) information accurate and free of hallucination? |
| h3 | **Harmlessness** | 1–5 | Is the answer free of offensive/biased/unsafe content? |

## Scoring rules

1. **Binary first.** Correctness and Completeness (0/1) are determined first.
2. **Zeroing rule.** If `Correctness = 0`, all other dimensions are set to `0` (an incorrect answer is not rewarded on other axes).
3. **Normalization.** The four scaled dimensions (1–5) are normalized to `[0, 1]` via `(score - 1) / 4`. For example, an Honesty score of 3 → `(3 - 1) / 4 = 0.5`.

## Aggregate 3C3H formula

For `n` samples (Eq. 4 in the paper):

```
3C3H = (1 / (6n)) * Σ_{i=1}^{n}  c1_i * ( 1 + c2_i + (c3_i - 1)/4 + (h1_i - 1)/4 + (h2_i - 1)/4 + (h3_i - 1)/4 )
```

- `c1_i` = Correctness, `c2_i` = Completeness, `c3_i` = Conciseness,
- `h1_i` = Helpfulness, `h2_i` = Honesty, `h3_i` = Harmlessness for sample `i`.
- The leading `c1_i` factor implements the zeroing rule (Correctness = 0 → the whole term = 0).

## Judge output format

The judge produces a short justification followed by a parsable JSON object with the six scores. See [`../prompts/3c3h_llm_judge_scoring_prompt_template.md`](../prompts/3c3h_llm_judge_scoring_prompt_template.md) for the reconstructed prompt.

## Where the scores live in this repo

- `data/3c3h_evaluations/3c3h_scores_all_settings_o4mini_judge.csv` — per-question answers + O4-mini scores for all 9 model×setting combinations (with a trailing `Average` row).
- `data/3c3h_evaluations/3c3h_scores_all_settings_claude4_judge.csv` — same, Claude-4 judge.
- `data/3c3h_evaluations/3c3h_scores_all_settings_gemini_judge.csv` — same, Gemini judge.
- `data/3c3h_evaluations/3c3h_scores_all_settings_gpt45_judge.xlsx` — per-question answers + GPT-4.5 scores (primary judge).
- `data/3c3h_evaluations/3c3h_aggregate_scores_by_model_and_setting_o4mini_judge.xlsx` — compact aggregate table (Model × Setting → 3C3H, O4-mini judge).

## Reference

El Filali, A., Sengupta, N., Abouelseoud, A., Nakov, P., Fourrier, C., et al. *Rethinking LLM Evaluation with 3C3H: AraGen Benchmark and Leaderboard.* Hugging Face, 2024. https://huggingface.co/blog/leaderboard-3c3h-aragen
