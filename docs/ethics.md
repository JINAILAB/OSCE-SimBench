# Ethics statement

## Synthetic-only patient personas

Every patient record produced by this pipeline — name, demographics,
vital signs, history, checklist, and standardised-patient script — is
**fictional**. There is no human-subjects data and no PHI in the
generated artefacts. The pipeline does not ingest, transform, or
release any real clinical encounter.

## Reference corpus provenance

The per-symptom reference text (`data/seed/<chief complaint>.txt`)
that drives stage 1 is supplied locally by the authors from a closed
Korean medical-education corpus. It is not redistributed in this
repository: see [`../data/README.md`](../data/README.md) for the
copyright reasoning and the bring-your-own-corpus protocol. Sample
files committed under `data/samples/` are placeholder text, not from
the closed corpus.

## Suitable and unsuitable uses

**Suitable** for:

- Training and evaluating LLM-based standardised-patient simulators.
- Benchmarking automated OSCE checklist scoring against reference
  rubrics.
- Augmenting Korean medical-dialogue corpora.
- Researching prompt-based synthetic-data generation for
  clinical-skills education.

**NOT suitable** for, and explicitly out of scope:

- Any form of clinical decision-making, diagnostic reasoning, or
  triage for real patients. The generated content is *educational
  simulation*, not clinical advice.
- Evaluation in non-Korean clinical, linguistic, or educational
  contexts. The corpus is Korean and the prompts encode
  Korean-OSCE conventions.
- Standalone medical training without supervisory oversight.
- Any use that contravenes
  [OpenAI's Usage Policies](https://openai.com/policies/usage-policies)
  governing the upstream models that produced the artefacts.

## Risks and mitigations

| Risk | Mitigation |
|---|---|
| LLM output may include clinically marginal or fabricated content. | Released as a research pipeline only; explicit "not for clinical decision-making" statement throughout README, dataset_card, and Croissant metadata. |
| The closed reference corpus is copyright-restricted. | The corpus itself is gitignored (`data/seed/`); only synthetic placeholder samples are committed. |
| Reproducibility under LLM stochasticity. | All three model endpoints are configured with `seed=42` (best-effort, per OpenAI's seed semantics) and decoding parameters are documented in the README "Models and decoding" table. |
| Misuse for high-stakes evaluation. | The dataset card and this document explicitly delimit suitable / unsuitable uses. |

## Compliance

- Code license: [Apache-2.0](../LICENSE).
- Generated artefacts (when separately distributed): see
  [`dataset_card.md`](dataset_card.md).
- Upstream model outputs are governed by OpenAI's
  [Usage Policies](https://openai.com/policies/usage-policies) and
  [Sharing & Publication Policy](https://openai.com/policies/sharing-publication-policy)
  in addition to the artefact license.

## Companion empirical-dialogue dataset

The empirical evaluation dataset (49 student × VSP dialogue sessions)
is released separately on HuggingFace under CC-BY-NC-SA-4.0 with its
own ethics statement and IRB-anchored anonymisation procedure. See
[`dataset_card.md`](dataset_card.md) for the link.
