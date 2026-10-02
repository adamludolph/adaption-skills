---
name: adaption
description: >
  Build, debug, review, or operate integrations using Adaption Labs APIs/SDK,
  including Invent, Adaptive Data, dataset ingestion/evaluation, AutoScientist
  training/alignment, monitoring, and exports. Use when the task explicitly involves
  Adaption or adaptionlabs.ai; not for generic ML or fine-tuning.
---

# Adaption Developer Integration

## Workflow router

| Intent | Path |
|---|---|
| Generate training data without a seed dataset | Invent; choose instruction or preference output and language expansion. |
| Transform existing examples | Adaptive Data; select mappings, output shape, recipes and controls. |
| Add examples or language/locale variants | Augment/translate/localize; distinct from full adaptation. |
| Preserve training-ready examples | Raw ingestion; prompt/completion or prompt/chosen/rejected mappings. |
| Train, align, select a model or inspect a recipe | AutoScientist; processed eligible data, supported model discovery/recommendations. |
| Discover resources, diagnose, evaluate or export | List/get/status/evaluation/download surfaces relevant to that resource. |

## Contract, observation, and defaults

Checked **2026-10-02** against public documentation. Recheck consequential calls
against the current endpoint and installed SDK; schemas, models and defaults change.
Distinguish **documented contract**, **reproducible/versioned observation**,
**attributed community observation**, and **hypothesis**. Record conflicting
method/version/request/response/time evidence; observations do not set API contracts.
[Field notes](references/field-notes.md) hold dated reports, research and conflicts.

Start account discovery with list/get/status and relevant pagination; correlate
existing dataset/run IDs before creating duplicates. Discovery does not authorize
a launch. Estimate the exact intended request before paid generation/adaptation/
expansion where supported; an estimate does not authorize a launch. Code review
or implementation alone does not authorize paid/live Adaption operations.
Imports can start processing/adaptation: use the deliberate raw/deferred path when appropriate.
Persist returned IDs; a client timeout stops waiting, not remote work. Retrieve
and resume the known resource before retrying creation or paying to repeat training.

## Critical Invent choices

Invent creates model-ready text training data from a natural-language specification,
without source/seed rows. Use `dataset_prompt`, requested `rows`, and current
`datasets.invent_domains()` if taxonomy is needed; do not freeze domain codes.
Choose `training_type="instruction_dataset"` for prompt/completion or
`"preference_pairs"` for chosen/rejected output. Translation expands languages;
localization adds country/language variants. Estimate the same request with
`estimate=True` before launch. Repeated `idempotency_key` returns the original
dataset. Poll asynchronously to `succeeded`/`failed`; distinguish requested rows
from observed `row_count`. Inspect output before export/training; do not adapt
Invent output again automatically. Do not infer tool traces, agent trajectories
or non-text generation from the research benchmark.
Details: [Invent](#invent-details); [research scope](references/field-notes.md#invent-research-scope-and-results).

## Critical Adaptive Data choices

Choose preservation (raw), transformation (adapt), expansion, or quality evaluation
deliberately; successful ingestion does not prove adaptation or quality evaluation.
Adaptive Data generates instruction or preference output from mapped source data.
Chat, per-row prompt/completion, shared prompt/context and image mappings have
different requirements. Safety controls **annotate, not filter**. Web grounding
and Blueprint guide generation; verify length controls with prompt rephrasing.
Language expansion can change rows/cost; omitted `language_expansion` preserves a
previous configuration, null clears it, and an object replaces it. Multimodal
context incurs higher pricing and currently disqualifies the dataset from
finetuning. Raw chosen/rejected ingestion is documented, but the older raw guide
conflicts: inspect processed shape/type before assuming alignment eligibility.
Details: [ingestion/mappings](#ingestion-and-mappings), [controls](#adaptive-data-controls-and-expansion).

## Critical AutoScientist choices

Use a processed eligible dataset; discover models and inspect recommended
hyperparameters before overriding. Dataset `training_type` is output shape;
run `training_method` is instruction/alignment; `hyperparams.training_type` is
LoRA/full strategy. Alignment requires preference data; read back the resolved
method because an ineligible request can resolve to instruction. Raw `data_format`
is chat/instruction encoding; adapted datasets ignore it.
Current iteration range is **1–5**. The target is an early-stop condition;
`succeeded` can also mean the budget ended without reaching it.
AutoScientist keys are **dataset-scoped and active-run-only**: repeating after
termination can start a new paid run. Retrieve the known run first.
Alignment's SFT→DPO handoff keeps one ID and can temporarily lack metrics/artifacts.
Download the **best iteration**, which may differ from the last.
Details: [launch](#autoscientist-launch), [diagnostics](#monitoring-and-evaluation-diagnostics),
[provenance](#launch-provenance-and-metric-interpretation), [downloads](#downloads).

## Invent details

[Guide](https://docs.adaptionlabs.ai/adaptive-data/invent-a-dataset/) and
[request reference](https://docs.adaptionlabs.ai/api/resources/datasets/methods/invent):
taxonomy can be inferred from the prompt; explicit domain/subdomain codes disable
inference. Use supported discovery, not a project's saved-taxonomy requirement.
An estimate creates no dataset and incurs no charge. Instruction output uses
enhanced prompt/completion fields; preference output adds chosen/rejected.
Source originals are absent for generated data; inspect the actual schema.

Language expansion selects translation languages or localization country/language
pairs and `sample_rate`. Validate supported values through current request
validation; do not copy a remembered language-count/code list into integrations.
Poll the returned dataset ID and record observed output, including shortfalls.

## Ingestion and mappings

Use the application's existing secret mechanism; the SDK reads `ADAPTION_API_KEY`.
[Authentication](https://docs.adaptionlabs.ai/introduction/getting-started/).

[Create reference](https://docs.adaptionlabs.ai/api/resources/datasets/methods/create):
local uploads return presigned **upload** instructions; a source URL can import
Hugging Face, Kaggle or Google Sheets. HF/Kaggle imports require selected files;
Sheets can select tabs and needs the supported public-link or connected-access
path. Source availability and supported formats differ; inspect that source's
schema rather than treating every URL/file as interchangeable.

The default processing mode is adapt. `defer_adaption=True` stores data awaiting
preprocessing; the documented start route is `POST /datasets/:id/start-adaption`.
Check SDK support before inventing a matching method name.
For already training-ready tabular data, use `processing_mode="raw"` and explicit
`column_mapping`: prompt plus completion **or** prompt plus chosen and rejected;
optional context is folded into prompts. Raw imports must share one format/schema.
Wait for processing and inspect the resulting columns and dataset type.
Current `datasets.create` reference documents raw processing for provider imports,
while the higher-level [creation guide](https://docs.adaptionlabs.ai/adaptive-data/create-a-dataset/)
still describes raw as local-upload-only. Remote raw import support is
documentation-conflicted; verify the current endpoint/SDK before relying on it.
The [raw guide](https://docs.adaptionlabs.ai/autoscientist/run-on-non-adapted-data/)
still says preference data requires adaptation; this conflicts with current create
support, so do not automatically transform raw pairs or promise DPO eligibility.

[Mapping guide](https://docs.adaptionlabs.ai/adaptive-data/select-columns):
map original headers at ingestion; downstream mappings use the processed schema.
Prompt/completion mapping can supply existing anchors for generation; context
supplies per-row grounding. `universal_prompt` is shared task text and needs a
per-row differentiator such as context or image.
Chat preserves conversation structure and excludes prompt/completion/context/
universal-prompt mappings; image may accompany it. Image cannot stand alone: it
needs text framing. Supported image encodings/paths depend on the import source;
a relative path supported by one source is not valid for every local upload.

## Adaptive Data controls and expansion

[Configuration guide](https://docs.adaptionlabs.ai/adaptive-data/configure-adaptive-data/):
choose output `training_type` (instruction or preference), `recipe_specification`
and `brand_controls` deliberately. Recipes cover deduplication, prompt rephrasing
and reasoning traces. `job_specification.max_rows` limits processed input;
sampling/deduplication/language expansion can change output counts. The guide
describes length via rewritten prompts, so check rephrasing when length has no
effect; chat disables prompt rephrasing. `brand_controls.length` offers
minimal/concise/detailed/extensive; Blueprint is freeform system guidance, not a
column mapping. `hallucination_mitigation` enables web-search grounding for text;
it is not proof of factual correctness.

[Adapt reference](https://docs.adaptionlabs.ai/api/resources/datasets/methods/adapt):
non-empty `safety_categories` enables analysis across all five categories,
not only the requested subset. Results annotate `prompt_safety_issues` and
`response_safety_issues`; rows remain. Preference preparation generates
chosen/rejected from the mapped prompt and optional completion, unlike raw
ingestion of existing pairs.

[Adapt SDK reference](https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/adapt):
`language_expansion` omission preserves saved settings, explicit null clears,
and an object replaces them. Translation uses languages; localization uses
country/language pairs; required `sample_rate` is 0.01–1. Credits depend on
expanded output; include the effective expansion in both estimate and launch.
Image mapping is automatically added to context. Multimodal context disqualifies
finetuning and attracts higher per-output-row pricing; inspect the exact estimate
and pricing flags rather than using text-only cost assumptions.

[Expansion guide](https://docs.adaptionlabs.ai/adaptive-data/expand-data/):
augment/translate/localize create a new dataset while preserving the source;
in-adapt language expansion occurs within that adaptation instead.
Augment adds curated domain/general examples; it excludes image datasets.
These methods support estimates and keys; wait on the new ID before export/train.
AutoScientist augmentation adds training rows without writing a new dataset.
[Quality evaluation](https://docs.adaptionlabs.ai/adaptive-data/evaluate-dataset-quality/)
is a distinct outcome, not a consequence proven by ingestion.

## Dataset state and counts

Track requested/source/processed/observed populations separately. An unexpected
minimum-row error requires current model/operation requirements and the actual
counted population before padding rows or blaming the UI. Deduplication,
expansion and partial output can affect usable rows.
API state, UI previews, quality evaluation and export availability can differ;
inspect the surface relevant to the operation. Read schemas rather than assuming
invented/adapted columns match source headers; use supported inference when
explicit mappings are unnecessary.

## AutoScientist launch

[Create reference](https://docs.adaptionlabs.ai/api/resources/autoscientist/methods/create):
discover via `autoscientist.list_models()`, prefer automatic model selection and
derived hyperparameters unless overrides serve the task.
[Recommend hyperparameters](https://docs.adaptionlabs.ai/api/resources/autoscientist/methods/recommend_hyperparams)
inspects the recipe without launching. Record effective augmentation counts.
Run `training_method` selects instruction/alignment; raw `data_format` controls
encoding; `hyperparams.training_type` selects lora/full. Confirm resolved values.

[Running guide](https://docs.adaptionlabs.ai/autoscientist/running-autoscientist/):
`target_win_rate` stops early; raising it does not guarantee better quality.
Compare best score with the target and budget. Threshold boundary acceptance has
an open documentation discrepancy in the field notes.
Keep AutoScientist's terminal-key behavior distinct from Invent's repeat-key
behavior; persist the resource ID and retrieve it before retrying creation.

## Monitoring and evaluation diagnostics

Public run states are pending/running/succeeded/failed/cancelled. Resume a known
run after a client wait timeout. Alignment spans SFT/DPO under the same run ID;
during handoff it can remain running without metrics or a downloadable artifact.
[Results guide](https://docs.adaptionlabs.ai/autoscientist/interpreting-results/).

Keep public status separate from internal/iteration/UI labels such as reported
`eval_failed`. Record run/dataset/iteration IDs, training evidence, each status,
timestamps and exact errors including null. Missing errors leave cause unresolved.
Inspect later recovery and whether a supported evaluation-only retry exists
before paying to repeat training; do not invent an endpoint. More training is a
reported workaround, not evidence that it repairs evaluation or improves quality.
Relevant hypotheses and dated diagnostics live in [field notes](references/field-notes.md).

## Launch provenance and metric interpretation

Record endpoint/SDK version, time and experiment label; dataset ID/origin/state and row
counts; submitted and resolved model, mappings, shape, encoding, method/strategy,
iteration budget, target, overrides and augmentation; key, returned run ID and
best-result configuration. These distinctions identify which Adaption operation
and artifact an experiment actually measured.

Separate Adaptive Data quality, application classification, AutoScientist win rate,
held-out/domain evaluation, leaderboard scores and local diversity/similarity.
Record population and iteration; delayed/missing evaluation is unresolved, not a
poor score. Optional comparison methods are in [field notes](references/field-notes.md).
Isolate necessary unsupported/community-client diagnostics, check response shape
and run/dataset attribution; internal fields are not supported API requirements.

## Downloads

[Dataset downloads](https://docs.adaptionlabs.ai/api/resources/datasets/methods/download)
stream processed rows: full output for ready datasets, partial successful rows for
failed datasets. Label partial recovery; it does not establish completed processing
or training eligibility. CSV/JSON/JSONL are text; Parquet is a compressed tar of
shards. Check resource/SDK labels rather than forcing one universal status enum.

[Model downloads](https://docs.adaptionlabs.ai/autoscientist/download-the-model/)
stream a compressed tar from the best iteration. Confirm successful completion
and `download_available`; alignment's artifact belongs to the DPO stage.
Neither download endpoint should be treated as a presigned-URL response.
Stream large artifacts rather than loading them into memory.
