# Tibbe-AG — Results Summary

All numbers below are taken directly from the paper and the evaluation files in this repository. No values have been estimated or invented.

## Table 1 — Response quality criteria (paper Table 1)

Only Tibbe-AG meets all four criteria.

| Criterion | Direct | RAG | Tibbe-AG |
| --- | :---: | :---: | :---: |
| Cites authentic sources | ✗ | ✓ | ✓ |
| Provides actionable specifics | ✗ | ✓ | ✓ |
| Includes scientific validation | ✗ | ✗ | ✓ |
| Includes clinical safety cues | ✗ | ✗ | ✓ |

## Table 2 — Average 3C3H scores across inference settings (paper Table 2)

`Mean` is the average across the four judge LLMs.

| Base Model | Method | GPT-4.5 | O4-mini | Claude-4 | Gemini | Mean |
| --- | --- | :---: | :---: | :---: | :---: | :---: |
| Qwen | Direct | 0.44 | 0.48 | 0.43 | 0.44 | 0.45 |
| Qwen | RAG | 0.62 | 0.62 | 0.75 | 0.76 | 0.69 |
| Qwen | Tibbe-AG | 0.73 | 0.74 | 0.85 | 0.86 | **0.80** |
| Mistral | Direct | 0.48 | 0.53 | 0.45 | 0.46 | 0.48 |
| Mistral | RAG | 0.65 | 0.66 | 0.76 | 0.77 | 0.71 |
| Mistral | Tibbe-AG | 0.76 | 0.77 | 0.86 | 0.87 | **0.82** |
| LLaMA-3 | Direct | 0.49 | 0.58 | 0.47 | 0.48 | 0.50 |
| LLaMA-3 | RAG | 0.67 | 0.70 | 0.78 | 0.79 | 0.73 |
| LLaMA-3 | Tibbe-AG | 0.77 | 0.79 | 0.88 | 0.89 | **0.83** |

## O4-mini judge — compact aggregate

From `data/3c3h_evaluations/3c3h_aggregate_scores_by_model_and_setting_o4mini_judge.xlsx` (matches the O4-mini column above; "Agentic (SciEval)" = Tibbe-AG):

| Model | No RAG | RAG | Agentic (SciEval) |
| --- | :---: | :---: | :---: |
| LLaMA-3 | 0.58 | 0.70 | 0.79 |
| Mistral | 0.53 | 0.66 | 0.77 |
| Qwen | 0.48 | 0.62 | 0.74 |

## Headline findings (from the paper)

- **Retrieval (RAG) improves factual accuracy by ~13%** over Direct inference.
- **The agentic self-critique adds a further ~10%** improvement, through deeper mechanistic insight and safety considerations.
- Gains are **consistent across all three base models and all four judges**, indicating the improvement is not tied to a specific evaluator.
- Example (LLaMA-3, GPT-4.5 judge): Tibbe-AG 0.77 vs. RAG 0.67 vs. Direct 0.49.

## Notes

- The three per-judge CSVs (`o4mini`, `claude4`, `gemini`) contain the per-question answers and the per-question scores for all nine model×setting combinations, plus a trailing `Average` row.
- The GPT-4.5 file is provided as `.xlsx` (`3c3h_scores_all_settings_gpt45_judge.xlsx`).
