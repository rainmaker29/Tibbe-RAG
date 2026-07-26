# Tibbe-RAG: From RAG to Agentic — Validating Islamic-Medicine Responses with LLM Agents

This repository accompanies the paper **"From RAG to Agentic: Validating Islamic-Medicine Responses with LLM Agents"** (Tibbe-RAG). It contains the curated Prophetic-medicine benchmark, the raw model generations for three inference settings, the multi-judge 3C3H evaluation outputs, qualitative examples, and documentation of the prompts and evaluation methodology.

- **Paper:** From RAG to Agentic: Validating Islamic-Medicine Responses with LLM Agents
- **arXiv:** [abs/2506.15911](https://arxiv.org/abs/2506.15911) · [PDF](https://arxiv.org/pdf/2506.15911) (cs.CL)
- **Venue:** Accepted at the **4th Muslims in Machine Learning (MusIML) Workshop**, co-located with the **42nd International Conference on Machine Learning (ICML 2025)**, Vancouver Convention Centre, Vancouver, Canada (July 2025).
- **Authors:** Mohammad Amaan Sayeed\*, Mohammed Talha Alam\*, Raza Imam, Shahab Saquib Sohail, Amir Hussain (\*equal contribution)
- **Affiliations:** Mohamed bin Zayed University of Artificial Intelligence (MBZUAI), UAE · VIT Bhopal University, India · Edinburgh Napier University, UK

> The full paper PDF is included at [`paper/`](paper/).

---

## Abstract

Centuries-old Islamic medical texts like Avicenna's *Canon of Medicine* and the Prophetic *Tibb-e-Nabawi* encode a wealth of preventive care, nutrition, and holistic therapies, yet remain inaccessible to many and underutilized in modern AI systems. Existing language-model benchmarks focus narrowly on factual recall or user preference, leaving a gap in validating culturally grounded medical guidance at scale. We propose a unified evaluation pipeline, **Tibbe-RAG**, that aligns 30 carefully curated Prophetic-medicine questions with human-verified remedies and compares three LLMs (LLaMA-3, Mistral-7B, Qwen2-7B) under three configurations: direct generation, retrieval-augmented generation, and a scientific self-critique filter. Each answer is then assessed by a secondary LLM serving as an agentic judge, yielding a single **3C3H** quality score. Retrieval improves factual accuracy by 13%, while the agentic prompt adds another 10% improvement through deeper mechanistic insight and safety considerations.

---

## The Tibbe-RAG framework

Tibbe-RAG is an agentic RAG pipeline grounded in classical Islamic medical knowledge (primarily *Tibb-e-Nabawi*). Given a health question `q`, it is evaluated under three inference settings:

| Setting | Description | Formulation |
| --- | --- | --- |
| **(E1) Direct** | Base LLM answers from the query alone, with no external grounding. | `A_f = LLM0(q)` |
| **(E2) RAG** | Prompt is augmented with top-k passages retrieved from the *Tibb-e-Nabawi* corpus. | `A_f = LLM0(q, R(q))` |
| **(E3) Tibbe-RAG (Agentic)** | Same retrieval as RAG, plus an explicit self-critique/validation prompt appended to refine the draft. | `A_f = LLM0(q, R(q), A0, q_val)` |

The agentic validation step directs the model to (i) fact-check each segment of the initial answer against the retrieved evidence, (ii) inject mechanistic context (e.g., ginger's effect on COX-2 pathways), and (iii) filter or flag unsafe recommendations (e.g., drug–herb interactions).

**Retrieval:** a dense retriever (ChromaDB) embeds both query and corpus passages into a shared space and returns the top-k passages by cosine similarity.

Only Tibbe-RAG satisfies all four response-quality criteria from the paper (Table 1): cites authentic sources, provides actionable specifics, includes scientific validation, and includes clinical safety cues.

---

## Repository structure

```
Tibbe-RAG/
├── README.md
├── paper/
│   └── Sayeed_etal_2025_TibbeAG_from_RAG_to_agentic_islamic_medicine_arXiv_2506.15911v2.pdf
├── data/
│   ├── benchmark/
│   │   ├── prophetic_medicine_30_question_benchmark_with_sources.csv
│   │   └── tibbe_nabawi_cures_from_quraan_and_rasulullaah_extracted_source_section.txt
│   ├── model_generations/
│   │   ├── llama3_8b_rag_answer_scientific_eval_source_verification.csv
│   │   ├── qwen2_7b_rag_answer_scientific_eval_source_verification.csv
│   │   ├── mistral_7b_rag_answer_scientific_eval_source_verification.csv
│   │   └── three_models_rag_vs_norag_answer_comparison.csv
│   └── 3c3h_evaluations/
│       ├── 3c3h_scores_all_settings_o4mini_judge.csv
│       ├── 3c3h_scores_all_settings_claude4_judge.csv
│       ├── 3c3h_scores_all_settings_gemini_judge.csv
│       ├── 3c3h_scores_all_settings_gpt45_judge.xlsx
│       └── 3c3h_aggregate_scores_by_model_and_setting_o4mini_judge.xlsx
├── examples/
│   ├── single_model_llama3_across_three_settings_example.csv
│   └── agentic_setting_across_three_models_example.csv
├── prompts/
│   ├── tibbe_ag_rag_generation_prompt_template.md
│   ├── tibbe_ag_agentic_scientific_validation_prompt_template.md
│   └── 3c3h_llm_judge_scoring_prompt_template.md
└── docs/
    ├── tibbe_ag_evaluation_methodology_3c3h.md
    └── tibbe_ag_results_summary_tables.md
```

---

## Dataset

A focused benchmark of **30 Prophetic-medicine question–answer pairs**, drawn from two classical sources:

1. *Cures from the Qur'aan and Rasulullaah* as presented in **5-Minute Madrasa in English** (Mufti A.H. Elias, 2012).
2. *Tibb-e-Nabawi (Medical Guidance & Teachings of Prophet Muḥammad)* (Dr. M. S. Shamsi, 2016).

**Curation (three steps):**
- **Section extraction** — the "(9) Cures from the Qur'aan and Rasulullaah" section was extracted from source 1 using a PyMuPDF-based script (raw extracted text preserved in [`data/benchmark/`](data/benchmark/)); Prophetic-medicine chapters were parsed from source 2.
- **Question generation & selection** — remedy descriptions were converted into questions ("What Prophetic remedy is recommended for `<ailment>`?"), giving ~120 candidates, then manually filtered to 30, balanced across five categories: nutritional therapies, herbal remedies, ritual supplications, hygiene practices, and wound treatments.
- **Representativeness & feasibility** — the 30 questions span spiritual and herbal dimensions while staying tractable for exhaustive evaluation (3 settings × 3 models × 3–4 judges). Each question carries its exact source (Sūrah, ḥadīth collection, or *Tibb-e-Nabawi* chapter/verse).

The benchmark file (`prophetic_medicine_30_question_benchmark_with_sources.csv`) has columns: `question`, `answer` (human-verified remedy), `source` (citation).

---

## Models and judges

**Base models (answer generators):**
- LLaMA-3 (Meta-Llama-3-8B-Instruct)
- Mistral-7B (Mistral-7B-Instruct-v0.2-GPTQ)
- Qwen2-7B (Qwen2-7B-Instruct-GPTQ-Int4)

**Judge models (3C3H scoring):**
- Primary judge: **GPT-4.5**
- Ablation judges: **O4-mini**, **Claude-4**, **Gemini**

---

## Main results

**Average 3C3H scores across inference settings** (from Table 2 of the paper; Mean is the average across the four judge LLMs):

| Base Model | Method | GPT-4.5 | O4-mini | Claude-4 | Gemini | Mean |
| --- | --- | --- | --- | --- | --- | --- |
| **Qwen** | Direct | 0.44 | 0.48 | 0.43 | 0.44 | 0.45 |
|  | RAG | 0.62 | 0.62 | 0.75 | 0.76 | 0.69 |
|  | Tibbe-RAG | 0.73 | 0.74 | 0.85 | 0.86 | **0.80** |
| **Mistral** | Direct | 0.48 | 0.53 | 0.45 | 0.46 | 0.48 |
|  | RAG | 0.65 | 0.66 | 0.76 | 0.77 | 0.71 |
|  | Tibbe-RAG | 0.76 | 0.77 | 0.86 | 0.87 | **0.82** |
| **LLaMA-3** | Direct | 0.49 | 0.58 | 0.47 | 0.48 | 0.50 |
|  | RAG | 0.67 | 0.70 | 0.78 | 0.79 | 0.73 |
|  | Tibbe-RAG | 0.77 | 0.79 | 0.88 | 0.89 | **0.83** |

Across all base models and judges, Tibbe-RAG (Agentic) consistently outperforms both Direct inference and standard RAG. Additional per-cell scores are in [`data/3c3h_evaluations/`](data/3c3h_evaluations/); a fuller breakdown is in [`docs/tibbe_ag_results_summary_tables.md`](docs/tibbe_ag_results_summary_tables.md).

---

## Qualitative examples

Two illustrative comparisons are provided in [`examples/`](examples/):

1. **`single_model_llama3_across_three_settings_example.csv`** — how a single model (LLaMA-3) answers the same question differently under No-RAG, RAG, and Agentic settings.
2. **`agentic_setting_across_three_models_example.csv`** — how the three models (LLaMA-3, Qwen, Mistral) differ from each other under the same (Agentic) setting.

---

## Prompts

The prompting strategy for each stage is documented in [`prompts/`](prompts/):

- `tibbe_ag_rag_generation_prompt_template.md` — retrieval-grounded generation prompt.
- `tibbe_ag_agentic_scientific_validation_prompt_template.md` — the self-critique / validation prompt (`q_val`).
- `3c3h_llm_judge_scoring_prompt_template.md` — the LLM-as-judge 3C3H scoring prompt.

> Note on provenance: the prompt templates are reconstructed from the descriptions in the paper (Section 2) and the official 3C3H documentation, together with the observed structure of the model outputs in this repository. They are documented as faithful scaffolds of the described method; replace them with the exact original prompts if/when the authors release them.

---

## Evaluation: 3C3H

Answers are scored with the **3C3H** measure introduced in the AraGen leaderboard (El Filali et al., 2024), which uses an LLM-as-judge across six dimensions — **C**orrectness, **C**ompleteness, **C**onciseness, **H**elpfulness, **H**onesty, **H**armlessness. See [`docs/tibbe_ag_evaluation_methodology_3c3h.md`](docs/tibbe_ag_evaluation_methodology_3c3h.md) for definitions, the scoring formula, and the zeroing rule.

Reference documentation: [Rethinking LLM Evaluation with 3C3H: AraGen Benchmark and Leaderboard](https://huggingface.co/blog/leaderboard-3c3h-aragen).

---

## What is included vs. not included

**Included:** the 30-question benchmark with sources, extracted source text, raw model generations for all three settings across the three base models, per-question 3C3H scores from multiple judges, aggregate score tables, qualitative examples, prompt documentation, and the paper PDF.

**Not included (would need to be added by the authors):** the runnable pipeline code — the ChromaDB indexing/retrieval scripts, the generation harness for the three base models, the judge-invocation/scoring scripts, and any environment/requirements files. The prompt files here document the method; they are not the original executable prompt code. If you have these artifacts, they can be dropped into a `src/` directory and referenced from this README.

---

## Citation

If you use this benchmark or the Tibbe-RAG framework, please cite the paper:

```bibtex
@inproceedings{sayeed2025tibbeag,
  title     = {From RAG to Agentic: Validating Islamic-Medicine Responses with LLM Agents},
  author    = {Sayeed, Mohammad Amaan and Alam, Mohammed Talha and Imam, Raza and Sohail, Shahab Saquib and Hussain, Amir},
  booktitle = {4th Muslims in Machine Learning (MusIML) Workshop, 42nd International Conference on Machine Learning (ICML)},
  year      = {2025},
  address   = {Vancouver, Canada},
  eprint    = {2506.15911},
  archivePrefix = {arXiv},
  primaryClass  = {cs.CL},
  url       = {https://arxiv.org/abs/2506.15911}
}
```

---

## Acknowledgements & sources

- Classical sources: *Tibb-e-Nabawi* (Dr. M. S. Shamsi, 2016) and *5-Minute Madrasa in English* (Mufti A.H. Elias, 2012).
- Evaluation measure: 3C3H / AraGen (Inception, MBZUAI, and Hugging Face).

> **Disclaimer:** This repository is for research purposes only. The remedies and answers herein are drawn from classical texts and LLM outputs and do **not** constitute medical advice. Consult a qualified healthcare professional before acting on any information here.
