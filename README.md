# OSCE Simulated-Patient Pipeline

**A Stratified Multi-Rater Evaluation of LLM-Based Virtual Standardized
Patients with a Deployed Data-Generation Platform**

Yongjin Yi · Jinyong Park · Sejoong Kim

Dankook University · Seoul National University

NeurIPS 2026 — Track on Evaluations and Datasets

[Dataset](https://huggingface.co/datasets/nwirandx/Korean-OSCE-history-taking-dialogues) ·
[Prompts](prompts/README.md) ·
[Quick start](#quick-start) ·
[Dataset card](docs/dataset_card.md) ·
[Citation](#citation)

## Overview

This repository provides a framework for authoring and evaluating Korean
OSCE (Objective Structured Clinical Examination) cases with large language
models. It includes two complementary components:

1. **Data generation** — turns per-symptom medical reference text into a
   structured corpus of fictional patients, OSCE-style interview checklists,
   and voice-ready standardised-patient scenarios.
2. **Runtime + auto-grading** — drives a simulated-patient chat from any of
   those scenarios, then auto-grades the student's session against the
   checklist and a five-dimension patient–doctor-relationship rubric.

The accompanying study combines a stratified human evaluation of virtual
standardized patients with dialogue collected on a deployed training
platform. These are distinct resources:

| Resource | Scope | Access |
|---|---|---|
| Static multi-rater evaluation | Four LLMs and a human-expert reference; 3 OSCE cases; 70 input prompts; 17 raters; 7,480 rating actions | Described in the paper and its appendices |
| Platform dialogue dataset | 49 sessions; 17 participants; 27 clinical scenarios; 1,763 question–answer pairs | [Hugging Face](https://huggingface.co/datasets/nwirandx/Korean-OSCE-history-taking-dialogues), CC-BY-NC-SA-4.0 |
| Code and prompts | Scenario authoring, a terminal-based patient simulation, grading, and Korean/English prompt templates | This repository, MIT |

The hosted dialogue dataset contains student questions and GPT-4o patient
responses. The closed reference corpus and full generated patient cohort
are documented in [`data/README.md`](data/README.md). The code here provides
the generation pipeline and terminal demo; see
[`docs/runtime.md`](docs/runtime.md) for the runtime interface.

## Evaluation summary

The study evaluates three sentence-level and six encounter-level metrics.
An 11-category history-taking taxonomy and three linguistic-form labels
help identify question-specific failure patterns. The human-expert
reference has the highest mean on all six encounter-level metrics.

For example, the paper reports the following overall-satisfaction scores
(Table 3; 1–5 Likert scale; 95% bootstrap confidence intervals). Each row
pools 51 ratings across three cases. These results describe the static
evaluation, rather than the automated scores in the platform dataset.

| Response source | Overall satisfaction | 95% CI |
|---|---|---|
| Human-expert reference | 4.61 | 4.41–4.78 |
| Claude 3.5 Sonnet | 3.82 | 3.53–4.12 |
| GPT-4o | 3.82 | 3.57–4.08 |
| Llama-3.1 70B | 2.71 | 2.35–3.06 |
| HyperCLOVA X | 3.88 | 3.65–4.10 |

## Data flow

```text
Reference text → Patient profiles → Interview checklists → Patient scripts
                                                               ↓
                                              Patient dialogue → Grading
```

See [`docs/pipeline.md`](docs/pipeline.md) for the four generation stages
and [`docs/runtime.md`](docs/runtime.md) for simulation and grading.

## Repository layout

```
.
├── src/
│   ├── prompts/                    Bilingual prompt module (ko.py, en.py)
│   ├── pipeline/                   Data-generation stages 1–4
│   ├── runtime/                    Simulator + interactive demo
│   ├── grading.py                  Auto-grader (per-item + relationship)
│   ├── utils.py, logging_utils.py
│   └── json_to_excel.py            Optional checklist export to .xlsx
├── prompts/                        Markdown documentation, one per prompt
│   ├── README.md                   Index of all 9 templates
│   ├── 01_generate_disease_list.md
│   ├── 02_generate_checklist.md
│   ├── 03_generate_scenario_draft.md
│   ├── 04_generate_scenario_polish.md
│   ├── 05_runtime_checklist.md     Runtime-side variant of the checklist prompt
│   ├── 06_chatbot_system.md
│   ├── 07_grading_checklist.md
│   ├── 08_grading_relationship.md
│   ├── 09_grading_feedback.md
│   └── en/                         Mirror of the above, in English
├── data/
│   ├── samples/                    Tiny example I/O committed to the repo
│   └── seed/                       gitignored — corpus + run outputs
├── docs/
│   ├── pipeline.md                 Data-generation flow
│   ├── runtime.md                  Runtime + grading flow
│   ├── dataset_card.md
│   └── ethics.md
├── croissant.json                  Croissant 1.0 + RAI metadata
├── run_all.sh                      bulk dataset generation
├── run_demo.sh                     interactive single-patient demo
├── requirements.txt
├── .env.example
└── LICENSE
```

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

## Quick start

```bash
git clone https://github.com/JINAILAB/osce-pipeline.git
cd osce-pipeline
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env       # then put your OpenAI key in .env
```

Run the terminal demo with the included sample patient:

```bash
bash run_demo.sh
```

Type `exit` to finish the encounter and generate the grading report.
The demo calls the model API using your configured key. Runtime options
are documented in [`docs/runtime.md`](docs/runtime.md).

## Use the dialogue dataset

The dataset can be loaded independently of the generation pipeline:

```python
from datasets import load_dataset

dialogues = load_dataset("nwirandx/Korean-OSCE-history-taking-dialogues", split="sessions")
session = dialogues.filter(lambda row: row["session_id"] == 564).sort("turn")
for turn in session:
    print("Student:", turn["question"])
    print("Patient:", turn["answer"])
```

This example uses the `datasets` package. The dataset card describes all
columns, English translations, and anonymisation. See
[`docs/dataset_card.md`](docs/dataset_card.md) for the relationship between
the released dialogues and the generated patient cohort.

## Bulk data generation

Place one UTF-8 text file per chief complaint under `data/seed/`, then run:

```bash
bash run_all.sh
```

A synthetic placeholder reference file is shipped at
[`data/samples/input/cough.txt`](data/samples/input/cough.txt) — copy it
into `data/seed/` for a single-symptom dry run.

## Running stages individually

```bash
python -m src.pipeline.stage1_process_disease_list
python -m src.pipeline.stage2_merge_json
python -m src.pipeline.stage3_generate_checklist [--test]
python -m src.pipeline.stage4_generate_scenario \
    [--symptom-limit N] [--disease-limit N] [--batch-size N]
```

## Citation

```bibtex
@inproceedings{yi2026vsp,
  title     = {A Stratified Multi-Rater Evaluation of LLM-Based Virtual
               Standardized Patients with a Deployed Data-Generation Platform},
  author    = {Yi, Yongjin and Park, Jinyong and Kim, Sejoong},
  booktitle = {Advances in Neural Information Processing Systems
               (NeurIPS 2026), Track on Evaluations and Datasets},
  year      = {2026}
}
```

## License

Code is released under the [MIT License](LICENSE). The seed corpus and
generated dataset are licensed separately — see
[`data/README.md`](data/README.md) and
[`docs/dataset_card.md`](docs/dataset_card.md). Ethics, suitable /
unsuitable uses, and risk mitigations are documented in
[`docs/ethics.md`](docs/ethics.md). Machine-readable Croissant 1.0 +
RAI metadata for the synthetic patient cohort is in
[`croissant.json`](croissant.json). Outputs of the upstream OpenAI
models (`gpt-4.1`, `gpt-4o`) are additionally governed by OpenAI's
[Usage Policies](https://openai.com/policies/usage-policies) and
[Sharing & Publication Policy](https://openai.com/policies/sharing-publication-policy).
