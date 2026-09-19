---
name: adaption
description: >
  Build, debug, review, and explain software integrations with the
  Adaption API and Python SDK. Use this skill for Adaption datasets,
  Adaptive Data, dataset invention, dataset preparation and augmentation,
  evaluation, AutoScientist model training, training experiments, model selection,
  experiment monitoring, checkpoint downloads, or when working with
  docs.adaptionlabs.ai. Do not trigger merely because a project uses
  generic machine learning or fine-tuning without involving Adaption.
---

# Adaption Developer Integration

Use this skill for Adaption-specific implementation, debugging, review, and
experimental work.

Official documentation:

https://docs.adaptionlabs.ai/

## Evidence model

Keep these evidence classes distinct:

- **Documented contract** — current official API/SDK documentation.
- **Verified behavior** — reproducible observation against a known API/SDK
  version or live endpoint.
- **Community observation** — useful diagnostic evidence, not a supported
  contract.
- **Hypothesis** — a proposed explanation that still requires testing.

Never promote a lower-confidence class into a higher one.

Treat current official documentation as authoritative over examples or remembered
constants in this skill. If official docs disagree internally, preserve the
conflict rather than silently choosing a value. Prefer the endpoint-specific
reference for request/schema constraints, then verify consequential differences
against the installed SDK or live API when practical. Record what was observed,
which version/endpoint produced it, and what remains uncertain.

Do not invent API methods, fields, status values, limits, model identifiers,
training modes, domain codes, or unsupported retry behavior.

For implementation, fit the host repository's existing language, dependency,
configuration, logging, and testing conventions. Prefer the official Python SDK
when it fits a Python project and documented REST APIs when direct HTTP is the
project's established pattern. Use `ADAPTION_API_KEY` or the project's existing
secret mechanism; never hard-code or expose a real key. Do not create an
Adaption-specific framework unless the task actually requires one.

## Choose the dataset path

Do not collapse Invent, Adaptive Data, and raw ingestion into one generic
pipeline.

### Invent from scratch

Use Invent when there is no source dataset and Adaption should generate a
training-ready dataset from a description.

Conceptually:

    description
        -> optional estimate
        -> Invent
        -> wait for completion
        -> inspect/evaluate/export as needed
        -> AutoScientist

A successfully completed invented dataset is already model-ready. Do not
automatically run Adaptive Data on it before AutoScientist unless another
transformation is intentionally requested.

Before an authorized Invent launch:

1. verify the current Invent request schema
2. discover current domain/subdomain codes through the documented API when
   taxonomy is used; do not hard-code remembered codes
3. use estimate mode when cost visibility is useful
4. keep the estimate materially equivalent to the intended launch
5. use documented idempotency support where a retry could duplicate paid work
6. record the returned dataset ID immediately
7. wait for a terminal dataset state before downstream use
8. record the observed completed `row_count`

Treat requested `rows` as a generation target, not proof of the final row count.
Preserve requested and observed counts separately. When the prompt alone is
sufficient, do not add taxonomy merely to make the request look more explicit.

### Adapt existing source data

Use Adaptive Data when ordinary/source data should be prepared, augmented, or
improved before training.

Conceptually:

    source data
        -> dataset ingestion
        -> source processing
        -> Adaptive Data
        -> wait for completion
        -> adapted dataset
        -> evaluation/export/training

Do not infer that successful ingestion means adaptation or its separate quality
evaluation has also completed.

### Use already training-ready data

Use raw processing only when the input is intentionally training-ready tabular
data and the current raw-ingestion requirements are satisfied. Current
documentation requires explicit prompt/completion mapping for raw ingestion;
verify the current schema before implementing.

Conceptually:

    training-ready tabular prompt/completion data
        -> raw ingestion
        -> wait for processing
        -> AutoScientist

Do not run Adaptive Data after raw ingestion unless the objective is to transform
that dataset rather than preserve the supplied training examples.

## Dataset state, counts, and mappings

Keep source/requested row counts separate from the API's completed `row_count`.
Processing, filtering, deduplication, partial completion, or other platform
behavior can make them differ.

When a minimum-row error appears inconsistent with a source file:

- inspect the dataset's current API state
- inspect its completed `row_count`
- check whether processing/deduplication changed usable rows
- verify the current minimum requirement
- do not classify the issue as a UI bug without evidence
- do not pad or mutate data merely to bypass the error before the counted
  population is understood

Backend/API state, application UI state, preview state, and export availability
can diverge. A dataset that exists or trains successfully through the API does
not prove every UI or export surface has synchronized. Report each surface
separately and use the relevant API resource state for API decisions.

Validate column mappings against the workflow actually being used. For raw
datasets, provide the mappings required by the current API. For adapted
datasets, do not guess generated column names. When AutoScientist can infer a
mapping and explicit control is unnecessary, prefer inference over invented
column names.

When prior account resources may matter, use documented read-only discovery
before asking for IDs or creating duplicates. Paginate list operations when a
complete relevant result set is required, correlate runs to dataset IDs, and
distinguish current account state from repository-local assumptions. Read-only
discovery does not authorize adaptation, training, cancellation, mutation, or
downloads.

## AutoScientist launch

Before creating a run:

1. confirm the dataset exists and required processing is complete
2. verify the current create-run schema
3. use only currently supported models and parameters
4. capture the exact launch configuration and returned experiment ID

Prefer automatic model selection unless a supported model override serves a
specific experimental objective. Prefer platform-derived hyperparameters unless
there is evidence-based reason to override them. Use the documented
recommend-hyperparameters operation when useful to inspect derived values
without launching training.

Treat `target_win_rate` as an early-stop threshold, not an optimization target.
A higher threshold can permit more of the allowed search budget; it does not
tell AutoScientist to "try harder" and does not guarantee the threshold will be
reached. Verify the current documented `max_iterations` range and
`target_win_rate` behavior for each consequential launch. Increase search
budget only when the experiment objective and cost envelope justify it.

Do not assume the final iteration is the best iteration. The AutoScientist loop
can revise the training recipe between iterations, so later results can
regress. Use documented best-result semantics. The downloadable model artifact
is the retained best trained artifact, not necessarily the final iteration.

Use documented idempotency behavior precisely. For AutoScientist, verify its
current scope and lifetime before relying on a retry key; do not assume a key
permanently deduplicates launches after a run reaches a terminal state.

## Evaluation and iteration diagnostics

Separate top-level AutoScientist run state from iteration-level, UI-only,
internal, or community-client states. An observed iteration status such as
`eval_failed` does not by itself establish failed training or a failed top-level
run.

For an evaluation failure, preserve:

- run ID and iteration ID when available
- dataset ID
- whether training itself completed
- top-level run status and iteration status separately
- exact error/error-message fields, including null
- relevant timestamps

Before paying to repeat training, inspect whether a later permitted iteration
recovered and whether the current supported API exposes an evaluation-only
retry. Additional iterations can sometimes provide another opportunity for an
intermittent evaluation to succeed, but that is resilience, not a fix for the
underlying problem and not evidence that more iterations improve model quality.

A null or absent evaluation error is an observability gap, not evidence for a
specific root cause. Do not promote a timeout, output cap, context limit,
generation failure, client bug, or similar explanation without direct evidence.

When one dataset repeatedly fails while comparable datasets do not, form narrow,
testable hypotheses around observable differences such as prompt length, target
length, format, mapping, or dataset structure. Change one suspected factor at a
time when practical.

## Experimental integrity and metrics

For comparative, research, or leaderboard-oriented work, record enough launch
provenance at submission time to reconstruct what actually ran. When applicable,
capture:

- SDK/package version or REST endpoint
- dataset ID, origin, processing/adaptation state, requested rows, and observed
  rows
- training format or data format when explicitly selected
- model choice or automatic selection
- column mapping
- `max_iterations` and `target_win_rate`
- training strategy/type and explicit hyperparameter overrides
- domain/general augmentation row counts
- idempotency key
- returned run/experiment ID
- timestamp and human-readable experiment label

Do not reconstruct a launch condition from memory when it could have been
recorded at submission.

For controlled comparisons, prefer the same underlying dataset/seed, change one
material variable at a time when practical, keep search budgets comparable,
replicate important findings when budget permits, and retain failed, cancelled,
and incomplete runs in the experiment record. One noisy run is exploratory
evidence, not a strong replicated conclusion.

Do not treat these signals as interchangeable:

- Adaptive Data quality evaluation
- application semantic/category classification
- AutoScientist on-dataset or best win rate
- held-out category/domain evaluation
- leaderboard normalized score
- external/local metrics such as diversity, similarity, or deduplication

Before optimizing a metric, establish what population and evaluation it
actually measures. Higher on-dataset win rate does not itself prove better
held-out performance or leaderboard score. Strong local diversity/similarity
does not establish correctness, relevance, or downstream training quality.

For held-out/domain or leaderboard-facing evaluations that are not fully
specified by the supported API, record whether the evaluation ran, which
iteration it evaluated, when it ran, and judged sample count when observable.
Do not interpret missing or delayed domain evaluation as poor model quality. If
assignment, timing, iteration selection, or sample count is nondeterministic,
do not use that metric as the sole pass/fail gate for a controlled comparison.

## Supported API versus observed behavior

Prefer the official SDK and documented REST API whenever they expose the needed
capability.

Community clients, UI behavior, and reverse-engineered/internal endpoints can be
useful evidence, but they are not supported contracts. If an unofficial path is
necessary:

- isolate it from the supported client path
- verify the live response shape before relying on it
- do not hard-code undocumented limits or statuses as permanent rules
- verify that a returned job/run is actually new before attributing it to an
  experimental condition
- record launch conditions locally when the remote record may omit them
- fail closed when attribution is ambiguous

When SDK docs, endpoint docs, UI state, and live responses disagree, preserve
the disagreement. Record the relevant version, endpoint/method, request,
HTTP status or exception, response/error payload, timestamp, and whether the
observation reproduced.

Treat reproducible live behavior as current operational evidence, not a
permanent API contract. Treat documentation as the intended supported contract,
not proof that every deployed surface currently behaves identically. Re-verify
historical observations such as row floors, concurrency limits, internal
statuses, stale-job behavior, or evaluation quirks before they influence
automation.

## Long-running, paid, and remote operations

Treat documented dataset processing, adaptation, evaluation, and AutoScientist
training as asynchronous. Store returned IDs and launch configuration, monitor
the relevant resource to a terminal state, expose useful errors, and use
reasonable timeout/resume behavior without silently duplicating creation.

Generating or reviewing integration code does not authorize paid or mutating
remote work. When a documented estimate mode exists for a material paid
operation, use it when useful for cost visibility; an estimate does not
authorize the real launch.

If the request is implementation-only, validate locally or with mocks where
appropriate and do not launch paid training/adaptation merely as a test. If the
user clearly authorizes actual execution and the environment has required
credentials/tools, execute only the requested workflow and do not silently
expand its scope.

For large artifacts, prefer streaming where supported and verify download
availability first. For AutoScientist, preserve the documented best-artifact
semantics rather than assuming the download represents the last iteration.
