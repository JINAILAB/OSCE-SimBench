# Runtime + auto-grading

The runtime side of the system runs an interactive OSCE session and
auto-grades it. All five LLM calls below use gpt-4o, seed 42, and run
through `src/runtime/llm_engine.py::LLMEngine`.

```
                ┌──────────────────────────────────────────────────────┐
                │                                                      │
   load patient │   PatientData (name, age, gender, chief_complaint,   │
   ─────────────►   disease, vital_sign, prompt)                       │
                │                                                      │
                └─────────────────────┬────────────────────────────────┘
                                      │
                                      ▼
                ┌──────────────────────────────────────────────────────┐
   student      │ chat_with_patient                                    │
   doctor       │ ── CHATBOT_SYSTEM_PROMPT                             │
   types ──────►│ ── temperature=0.5, streaming=true                   │
   questions    │                                                      │
                │ Each turn streamed back as the patient response.     │
                │ Conversation history kept in LLMEngine.memories.     │
                └─────────────────────┬────────────────────────────────┘
                                      │ student types `exit`
                                      ▼
                ┌──────────────────────────────────────────────────────┐
                │ generate_checklist                                   │
                │ ── CHECKLIST_PROMPT_TEMPLATE                         │
                │ ── temperature=0                                     │
                │ Produces a fresh OSCE checklist with                 │
                │ {question, purpose, order} items.                    │
                └─────────────────────┬────────────────────────────────┘
                                      │
                                      ▼
                ┌──────────────────────────────────────────────────────┐
                │ evaluate_checklist                                   │
                │ ── GRADING_CHECKLIST_ASSISTANT_PROMPT                │
                │ Adds asked: 0|1 to every checklist item.             │
                │                                                      │
                │ evaluate_relationship                                │
                │ ── GRADING_SCORE_ASSISTANT_PROMPT                    │
                │ 5 dimensions × 1–5 Likert with reasons.              │
                │                                                      │
                │ generate_feedback                                    │
                │ ── GRADING_FEEDBACK_ASSISTANT_PROMPT                 │
                │ ≤350 word three-section student-facing report.       │
                └─────────────────────┬────────────────────────────────┘
                                      │
                                      ▼
                ┌──────────────────────────────────────────────────────┐
                │ GradingResult                                        │
                │   checklist_score    : <% of items asked>            │
                │   relationship_mean  : <mean of 5 Likert scores>     │
                │   total_score        : <0..100>                      │
                │   grade              : A | B | C | D | F             │
                │   feedback           : str                           │
                └──────────────────────────────────────────────────────┘
```

## Score formula

```
total_score = (checklist_sum + relationship_sum) / (n_items + n_relations) * 100
```

with `n_relations = 5`. The letter grade follows: ≥80 A, ≥70 B, ≥60 C,
≥50 D, else F.

## Models and decoding

| Component | Model | Temperature | Seed |
|---|---|---|---|
| Data-gen stages 1, 3, 4 | gpt-4.1 | 0.2 | 42 |
| Runtime: checklist / scenario / grading | gpt-4o | 0 | 42 |
| Runtime: patient role-play | gpt-4o | 0.5 | 42 |

These are the defaults in the public code. The released platform dataset
records the dated model `gpt-4o-2024-11-20`, while the terminal demo uses
the `gpt-4o` alias. Consult the
[dataset card](https://huggingface.co/datasets/nwirandx/Korean-OSCE-history-taking-dialogues)
for the collection configuration. Seeded calls provide best-effort
reproducibility; the alias does not pin the historical model snapshot.

## Why two different OpenAI models?

The runtime + grading path uses `gpt-4o`; the data-generation pipeline
uses `gpt-4.1`. Both are kept as-run because the experiment evolved in
two phases — the gpt-4o-based simulator was built and validated first,
and the bulk dataset was generated later under gpt-4.1 once the dataset
schema had stabilised. We do not retroactively switch the models, since
that would invalidate the calibration of the grader's relationship
rubric (which was tuned against gpt-4o transcripts).

## Demo

```bash
bash run_demo.sh                              # Korean (default)
bash run_demo.sh --lang en                    # English
bash run_demo.sh --patient mypatient.json     # custom patient JSON
```

The demo loads `data/samples/patient/example_patient.json` by default,
which is a 39-year-old female with rheumatoid arthritis (`손마디가 아파요`
chief complaint). On `exit`, the auto-grading pipeline runs and prints
the result.
