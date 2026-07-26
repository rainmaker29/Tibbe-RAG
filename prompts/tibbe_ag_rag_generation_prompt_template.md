# Tibbe-AG — RAG Generation Prompt Template

**Stage:** Initial answer generation (`A0`) in the RAG (E2) and Tibbe-AG (E3) settings.

**Provenance:** Reconstructed from the paper's Methodology (Section 2, "Initial Answer") and the observed structure of the RAG answers in `data/model_generations/`. This is a faithful scaffold of the described method, not a verbatim copy of the authors' original prompt.

---

## What the paper specifies

From Section 2 (Initial Answer, `A0`):

> "Under the hood, `LLM0` is prompted with a structured template that interleaves user question, document excerpts, and explicit 'extract-and-summarize' instructions. Internally, it attends over each passage's provenance tokens to align outputs with source assertions."

So the prompt must:
1. Present the user question `q`.
2. Interleave the top-k retrieved passages `R(q)` from the *Tibb-e-Nabawi* corpus, each with its provenance/citation metadata.
3. Instruct the model to **extract and summarize** actionable recommendations grounded strictly in the retrieved passages, preserving direct textual traces (for traceability and citation).

---

## Reconstructed template

```
System:
You are a scholar of Prophetic Medicine (Tibb-e-Nabawi). Answer the user's
health question using ONLY the retrieved passages provided below. Extract the
relevant remedy, summarize it into clear, actionable guidance (including
quantities/dosage and duration when stated in the source), and cite the exact
source/reference for every claim. If the passages do not contain an answer, say
so rather than inventing one.

User:
Question: {question}

Retrieved passages from Tibb-e-Nabawi:
[1] {passage_1_text}   (source: {passage_1_citation})
[2] {passage_2_text}   (source: {passage_2_citation})
...
[k] {passage_k_text}   (source: {passage_k_citation})

Instructions:
- Extract the remedy/remedies relevant to the question.
- Summarize into actionable steps (ingredients, quantity, frequency, duration).
- Preserve and cite the source reference for each recommendation.
- Do not add information that is not supported by the passages above.
```

**Direct (E1) setting:** the same generation step is run **without** the retrieved-passages block — the model receives only `Question: {question}`.

---

## Notes

- `k` (number of retrieved passages) and the embedding model are set by the ChromaDB dense retriever; the exact values are not specified in the paper text.
- The RAG answers in this repo frequently open with a greeting ("Assalamu alaikum, …") and cite the source book/point numbers, consistent with the extract-and-cite instruction above.
