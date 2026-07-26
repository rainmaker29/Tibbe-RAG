# Tibbe-AG — Agentic Scientific Validation Prompt Template (`q_val`)

**Stage:** Refinement step (`A_f`) in the Tibbe-AG (E3) / Agentic setting.

**Provenance:** Reconstructed from the paper's Methodology (Section 2, "Refinement") and the observed structure of the `*_scientific_eval` columns in `data/model_generations/`. This is a faithful scaffold of the described method, not a verbatim copy of the authors' original prompt.

---

## What the paper specifies

From Section 2 (Refinement, `A_f`):

> "To guard against hallucinations and to enrich mechanistic and safety rationale, we append an explicit validation prompt `q_val` to the base LLM's input."
>
> The agentic step performs three sub-tasks: it directs the model to **(i)** fact-check each `A0` segment against `R(q)`, **(ii)** inject mechanistic context (e.g., ginger's effect on COX-2 pathways), and **(iii)** filter or flag unsafe recommendations (e.g., drug–herb interactions).

The same base LLM re-evaluates its own draft `A0` against the retrieved evidence and applies mechanistic + safety checks.

---

## Reconstructed template

```
System:
You are now acting as a medical scientist reviewing a draft answer that was
produced from classical Islamic-medicine texts. Critically validate the draft
against the retrieved evidence and scientific knowledge.

User:
Question: {question}

Retrieved passages (evidence):
[1] {passage_1_text} (source: {passage_1_citation})
...
[k] {passage_k_text} (source: {passage_k_citation})

Draft answer (A0) to validate:
{initial_answer_A0}

Validation instructions (q_val):
1. Scientific accuracy — fact-check every claim in the draft against the
   retrieved evidence and established science. Flag anything unsupported.
2. Mechanistic context — where a remedy is plausible, explain the biological
   mechanism (e.g., active compound and its pathway) and cite supporting
   evidence where possible.
3. Safety — identify contraindications, drug–herb interactions, unsafe
   dosages, and populations at risk; recommend professional consultation
   where appropriate.

Produce a refined, evidence-anchored final answer that keeps the authentic
source citation, adds the mechanistic and safety reasoning, and removes or
clearly flags any unsafe or unsupported recommendation.
```

---

## Notes

- The refinement is formally: `A_f = LLM0(q, R(q), A0, q_val) = LLM0(q, R(q), LLM0(q, R(q)), q_val)` — i.e., the **same** base model is called twice.
- The `*_scientific_eval` and `*_source_verification` columns in the model-generation files reflect the two facets of this validation (scientific critique and source verification).
- The Agentic answers in this repo characteristically contain sections such as "Scientific Accuracy", "Safety Considerations", and interaction/contraindication warnings, consistent with the three sub-tasks above.
