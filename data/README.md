# Data

This directory documents what does and does not ship with the public repo.

## What is committed

Under `samples/`:

- `samples/input/cough.txt` — synthetic placeholder reference file.
  **Not** drawn from any copyrighted textbook. Lets a reviewer drive a
  single-symptom data-gen dry run end-to-end without the full seed
  corpus.
- `samples/output/sample_diseases.json` — one symptom × one patient,
  trimmed from the stage 1 output.
- `samples/output/sample_with_checklist.json` — same patient with the
  stage 3 checklist attached.
- `samples/output/sample_with_scenario.json` — same patient with the
  stage 4 polished `prompt` field.
- `samples/patient/example_patient.json` — a self-contained patient
  record used by the interactive runtime demo
  (`python -m src.runtime.main`). Not part of the bulk dataset.

## What is gitignored

Everything under `seed/`. This includes:

- The per-symptom reference chapters (`seed/<chief complaint>.txt`) used
  during the experiment.
- All generated artifacts produced by `run_all.sh`:
  - `seed/results/<symptom>_diseases.json` (stage 1)
  - `seed/all_diseases.json` (stage 2)
  - `seed/all_disease_with_checklists.json` (stage 3)
  - `seed/all_disease_with_scenarios.json` (stage 4, final)
  - `seed/checklist_excel/` (optional Excel export from `json_to_excel.py`)

## Why the seed corpus is excluded

The reference chapters used in the experiment are derived from medical
education material whose redistribution is restricted. Releasing them in
this repo would be a copyright concern. The pipeline itself is corpus-
agnostic — see "Bring your own corpus" below.

## How to obtain the released dataset

The empirical dialogue dataset (49 sessions, 17 participants, 27 scenarios,
and 1,763 question–answer pairs) is available on
[Hugging Face](https://huggingface.co/datasets/nwirandx/Korean-OSCE-history-taking-dialogues)
under CC-BY-NC-SA-4.0. This release contains student questions and virtual
patient responses; it does not contain the full generated patient cohort
or the closed reference corpus. See
[`../docs/dataset_card.md`](../docs/dataset_card.md) for provenance and
license details.

## Bring your own corpus

To run the data-generation pipeline on your own data:

1. For each chief complaint, write a single UTF-8 text file containing
   reference knowledge (textbook chapter, clinical-guideline excerpt,
   review article, etc.) and place it at:

   ```
   data/seed/<chief complaint>.txt
   ```

   The file's stem becomes the `symptom` label in every downstream
   artifact. Use Korean for the file name if you want the generated data
   to remain Korean — the default prompts are Korean.

2. Make sure `OPENAI_API_KEY` is set (`cp .env.example .env` and edit).

3. From the repository root, run:

   ```bash
   bash run_all.sh
   ```

The pipeline iterates over every `.txt` file directly under `data/seed/`.
There is no other configuration; the file list is derived by glob.

## Running the runtime demo

Independent of the bulk pipeline, the runtime simulator + grader can be
exercised on a single patient JSON:

```bash
bash run_demo.sh                                          # KO, default patient
bash run_demo.sh --lang en                                # EN, default patient
bash run_demo.sh --patient data/samples/patient/foo.json  # custom patient
```

A patient JSON must validate against `src.runtime.schemas.PatientData`
(name, age, gender, chief_complaint, disease, vital_sign, prompt).
