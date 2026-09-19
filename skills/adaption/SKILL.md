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

Use this skill whenever the task involves implementing, debugging,
reviewing, or explaining an Adaption integration.

Official documentation:

https://docs.adaptionlabs.ai/

## Core principles

- Inspect an existing repository before choosing architecture.
- Integrate with the project's current conventions.
- Build the smallest correct solution.
- Do not create an Adaption-specific framework unless the user
  explicitly requests one or the host project's architecture requires it.
- Verify exact API behavior against the current official documentation.
- Treat current official documentation as authoritative over examples
  contained in this skill.
- If current official documentation disagrees internally, do not silently
  choose a value. Prefer the endpoint-specific API reference for schema
  constraints, then verify against the installed SDK or live validation
  when the difference is consequential. Report the conflict and the
  evidence used.
- Never invent API methods, request fields, response fields, status
  values, model identifiers, or training modes.

## Determine the task

Classify the request as one or more of:

- explain an API or workflow
- review existing integration code
- debug an integration
- create/import/upload a dataset
- invent a dataset from scratch
- run Adaptive Data
- augment or otherwise prepare a dataset
- evaluate a dataset
- export/download dataset output
- create an AutoScientist run
- monitor an AutoScientist run
- cancel a run
- inspect available models
- inspect recommended hyperparameters
- download a trained checkpoint
- build a CLI, service, application, or library using these capabilities

Do not modify files for a purely informational question unless the user
asks for implementation.

## Inspect the repository

Before implementing code, identify:

- language and runtime
- dependency/package manager
- application architecture
- existing API/client abstractions
- environment/configuration conventions
- logging conventions
- synchronous versus asynchronous style
- test framework
- formatter/linter/type-check commands

Reuse those conventions.

## Documentation verification

Before writing exact Adaption calls, consult the relevant current
documentation.

Pay particular attention to:

- SDK version requirements
- method signatures
- request and response fields
- supported dataset formats
- processing modes
- asynchronous status behavior
- AutoScientist parameters
- downloadable artifact conditions

Do not copy large sections of the documentation into the repository.
Link to the official documentation where appropriate.

## Authentication

Prefer:

    ADAPTION_API_KEY

or the host project's established secret-management mechanism.

Never:

- hard-code a real key
- print a key
- log a key
- commit a key
- include a real key in tests or fixtures

For Python SDK applications, prefer normal environment-based client
initialization when consistent with current documentation.

## Choose the dataset workflow

Choose among three distinct paths. Do not collapse them into one generic
dataset pipeline.

### Invent from scratch

Use dataset invention when there is no source dataset and the user wants
Adaption to generate a model-ready dataset from a description.

Conceptually:

    dataset description
        -> optional estimate
        -> Invent
        -> wait for completion
        -> inspect/download/evaluate as appropriate
        -> AutoScientist

An invented dataset that has completed successfully is already
model-ready. Do not automatically run Adaptive Data on it before
AutoScientist unless the user explicitly asks for another transformation.

### Adapt existing source data

Use Adaptive Data for ordinary/source data that should be prepared or
improved before model training.

Conceptually:

    source data
        -> dataset ingestion
        -> wait for processing
        -> Adaptive Data
        -> wait for completion
        -> adapted dataset
        -> evaluation/export/training

### Use already training-ready data

Use raw processing only when the input is intentionally already
training-ready tabular prompt/completion data and the current API
requirements for raw processing are satisfied.

Conceptually:

    training-ready prompt/completion data
        -> raw dataset ingestion
        -> wait for processing
        -> AutoScientist

Do not call Adaptive Data after creating a raw dataset unless the user
explicitly asks to change that workflow.

## Invented datasets

Before an authorized Invent launch:

1. verify the current `datasets.invent` request schema
2. fetch current domain/subdomain codes with `datasets.invent_domains()`
   when explicit taxonomy will be used; do not hard-code remembered codes
3. use `estimate=True` when cost visibility is useful before generation
4. keep the estimated request materially identical to the intended launch
5. use a documented idempotency key when retries could duplicate work
6. store the returned dataset ID immediately
7. wait for a terminal dataset state before downstream use
8. record the completed dataset's observed `row_count`

Treat requested `rows` as a generation target, not proof of the final
row count. For provenance, keep both the requested row count and the
observed completed `row_count`.

When the prompt alone is sufficient, do not add domains merely to make
the request look more explicit. When domains or subdomains matter to the
task, use only current codes returned by the API.

### Row counts and surface-state mismatches

Keep source/requested counts separate from the API's completed
`row_count`. Processing, filtering, deduplication, partial completion,
or other platform behavior can make them differ.

When a minimum-row error appears to contradict the source file's row
count:

- inspect the dataset's current processing state
- inspect the completed API `row_count`
- check whether processing or deduplication reduced usable rows
- verify the current minimum-row requirement
- do not immediately classify the problem as a UI bug
- do not pad or mutate the dataset merely to bypass the error until the
  actual counted population is known

Also distinguish backend resource state from UI and export visibility.
A dataset that exists or trains successfully through the API does not by
itself prove that every UI, preview, Hugging Face export, or other
surface has synchronized. Report those states separately and use the
relevant API resource state for API decisions.

## Column mappings

Validate mappings against the appropriate schema.

For raw datasets, provide the mappings required by the current API.

For adapted datasets, do not guess generated column names.

When AutoScientist supports inferring a mapping and the developer does
not need explicit control, prefer inference over invented column names.

## Existing account state

When prior Adaption resources may be relevant to the task, prefer
documented read-only discovery before asking the developer to manually
provide identifiers or creating duplicate resources.

When appropriate:

- list existing datasets
- list existing AutoScientist runs
- follow pagination until the relevant result set is exhausted
- correlate runs with their dataset IDs
- use documented get/detail operations when more information is needed
- distinguish current account state from repository-local assumptions

Read-only discovery does not authorize mutation, cancellation, training,
adaptation, or artifact downloads.

Do not retrieve broad account history when it is unrelated to the
developer's request.

## AutoScientist

Before creating a run:

1. confirm the dataset exists
2. confirm required dataset processing has completed
3. inspect current create-run documentation
4. use only documented parameters

Prefer automatic model selection unless the developer specifically needs
a supported model override.

Prefer platform-derived hyperparameters unless the developer has a
specific reason to override them.

When useful, inspect recommended hyperparameters before launching a run.

Do not invent a generic "method" or alignment mode that is not exposed
by the current API.

### Iterations and early stopping

Treat `target_win_rate` as an early-stop threshold, not as an optimization
objective. Raising it allows AutoScientist to continue searching through
more of the permitted iterations; it does not instruct the optimizer to
"try harder" or guarantee that win rate.

Verify the current documented `max_iterations` range and target-win-rate
semantics before launching. For an explicitly authorized leaderboard or
maximum-search run where additional iteration cost is acceptable, a high
early-stop threshold such as `0.95` together with the currently supported
maximum iteration count may be appropriate. Do not silently choose that
more expensive search configuration for ordinary runs.

Do not assume the final iteration is the best iteration. Use the current
documented best-result and checkpoint semantics.

### Evaluation and iteration failure diagnostics

Separate the top-level AutoScientist run state from iteration-level,
UI-only, internal, or community-client statuses. In particular, do not
automatically equate an observed `eval_failed` iteration with failed
training or a failed top-level run unless the current API evidence shows
that relationship.

For an evaluation failure:

1. capture the run ID and iteration ID when available
2. record whether training itself completed
3. record the top-level run status and iteration status separately
4. capture any error/error-message field exactly, including null
5. preserve timestamps and the dataset ID
6. inspect whether a later permitted iteration recovered before deciding
   that the entire run must be relaunched
7. verify whether the current API exposes an evaluation-only retry before
   paying to repeat training or augmentation

More permitted iterations can sometimes provide another chance for an
intermittent evaluation to succeed, but that is a resilience possibility,
not a fix for the underlying failure and not evidence that more iterations
improve model quality. Extra iterations consume time and potentially
credits.

The AutoScientist loop explicitly revises the training recipe between
iterations. Later iterations can therefore be worse than earlier ones.
Compare per-iteration results when they are available, preserve the best
iteration semantics, and never describe a later regression as evidence
that the retained best checkpoint also regressed without verifying it.

When one dataset repeatedly fails evaluation while comparable datasets
do not, form narrow, testable hypotheses about dataset properties such as
prompt length, target length, format, or other observable differences.
Change one suspected factor at a time when practical. Do not promote a
generation timeout, output cap, context limit, or similar mechanism to
root cause without direct evidence.

A null or missing evaluation error is an observability gap, not evidence
for a specific client-side cause.

### AutoScientist experimental integrity

For comparative, research, or leaderboard-oriented work, preserve enough
provenance to reconstruct exactly what was launched.

Record at submission time, when applicable:

- SDK/package version or REST API path used
- dataset ID, requested row count when relevant, and observed row count
- dataset origin and processing/adaptation state
- invented/adapted dataset training format when relevant, such as
  `instruction_dataset` versus `preference_pairs`
- model or automatic-model-selection choice
- AutoScientist `data_format` when explicitly set
- `max_iterations`
- `target_win_rate`
- AutoScientist training strategy/type when explicitly set, such as
  `lora` versus `full`
- domain and general augmentation row counts
- explicit hyperparameter overrides
- column mapping
- idempotency key
- returned run/experiment ID
- timestamp and human-readable experiment label

Do not infer a launch condition later when the original request can be
recorded at submission time.

When testing a change:

- prefer paired comparisons on the same underlying dataset/seed
- change one material experimental variable at a time when practical
- keep search budgets and iteration counts comparable
- replicate important results when budget permits
- retain failed, cancelled, and incomplete runs in the experiment record
- distinguish exploratory evidence from replicated evidence

Do not draw a strong conclusion from one noisy run when a paired or
replicated comparison is practical.

### Metric separation

Do not treat these as interchangeable signals:

- Adaptive Data quality evaluation
- dataset semantic/category classification shown by the application
- AutoScientist on-dataset or best win rate
- held-out category/domain evaluation
- leaderboard normalized score
- external or local dataset metrics such as diversity, similarity, or
  deduplication scores

Before optimizing a metric, verify what population and evaluation it
actually measures. A higher on-dataset win rate does not by itself prove
better held-out category performance or a better leaderboard score.
Likewise, a strong local diversity or similarity metric does not by
itself establish correctness, relevance, or downstream training quality.

For held-out category/domain evaluations or leaderboard-facing
evaluations that are not fully specified by the supported API, record
whether the evaluation ran, which iteration it evaluated, when it ran,
and the judged sample count when those fields are observable. Do not
treat a missing or delayed domain evaluation as evidence of poor model
quality. If evaluation assignment, timing, iteration selection, or sample
count is nondeterministic, do not use that metric as the sole pass/fail
gate for a controlled comparison; report the missingness or mismatch.

### Unofficial or empirically observed behavior

Community clients and reverse-engineered endpoints can reveal useful
platform behavior, but they are evidence to verify rather than supported
API contracts.

Prefer the official SDK and documented REST API whenever they provide the
required capability.

If an unofficial/internal endpoint is necessary:

- isolate that use from the normal supported client path
- verify the live response shape before relying on it
- do not hard-code undocumented limits or statuses as permanent rules
- verify that a returned job/run is actually new before attributing it to
  an experimental condition
- record launch conditions locally because internal job records may not
  preserve every experimental parameter
- fail closed when attribution is ambiguous

When the SDK docstring, API reference, application UI, and observed live
response disagree, preserve the disagreement instead of collapsing it
into one claim. Record, when available:

- SDK/package version
- documentation URL or section
- endpoint or SDK method
- request parameters
- HTTP status or exception type
- response/error payload
- timestamp
- whether the observation was reproduced

Treat reproducible live behavior as current operational evidence, not as
a permanent API contract. Treat documentation as the intended supported
contract, not proof that every deployed surface currently matches it.

Historical observations such as row-count floors, concurrency caps,
additional internal statuses, evaluation behavior, or stale-job behavior
should be re-verified against current platform behavior before they
influence automation.

## Long-running operations

Treat dataset processing, adaptation, evaluation, and training as
asynchronous when documented. Do not assume that a completed adaptation
means its separate quality evaluation is already complete.

A robust integration should:

1. store returned dataset/run/experiment IDs
2. store the launch configuration needed for later attribution
3. monitor status
4. recognize successful terminal states
5. recognize failure/cancellation states
6. expose useful error details
7. support reasonable timeout behavior
8. avoid duplicate creation when execution is retried
9. support resume behavior when appropriate for the host application

Use documented idempotency support for retryable creation operations
where available. Verify each endpoint's idempotency scope and lifetime;
do not assume that a key permanently deduplicates requests after a
resource reaches a terminal state.

## Cost and remote execution

Generating integration code does not automatically authorize unnecessary
paid API operations.

When a documented estimate mode exists for a material paid operation,
prefer using it before an authorized launch when doing so helps the user
understand cost without creating work. An estimate does not itself
authorize the real launch.

If the user asks to implement support but does not ask for a real
training/adaptation run:

- implement it
- validate locally
- mock where appropriate
- do not launch paid work merely as a test

If the user clearly requests actual execution and the environment has
the required credentials and tools, execute the requested workflow.

Do not silently expand the scope of remote operations.

## Downloads

For potentially large dataset or model artifacts:

- prefer streaming to disk
- validate successful status first
- verify that a download is available
- avoid unnecessary in-memory buffering
- propagate download failures clearly

For AutoScientist, treat the downloadable checkpoint according to the
current documented semantics rather than assuming it represents the
last iteration.

## Python versus REST

For Python repositories, prefer the official SDK when it fits the
existing architecture.

For other languages, or when direct HTTP is already the project's
established pattern, use the current documented REST API.

Do not add Python merely because the Python SDK exists if the surrounding
project is implemented in another language.

## Verification

After implementation, run the relevant existing project checks:

- focused tests
- broader tests when appropriate
- formatter
- linter
- type checker

Repair failures caused by your changes.

Distinguish between:

- static/local verification
- mocked API verification
- live Adaption API verification

Never claim live validation when no live operation was performed.

## Debugging existing integrations

When debugging:

1. inspect the actual request/code
2. inspect the actual API response/error when supplied
3. verify the relevant current documentation
4. compare the integration against documented requirements
5. identify the smallest likely correction
6. modify code only when requested or clearly part of the task

Do not mask server errors with speculative client-side explanations.

## Greenfield requests

If the developer explicitly asks for a new:

- CLI
- library
- service
- application
- automation script

create the smallest conventional project structure that satisfies that
request.

The rule against creating a framework does not prohibit building a new
application when the user explicitly wants one.

## Completion response

After implementation, report concisely:

- what was implemented
- files changed
- Adaption workflow/API operations involved
- tests/checks run
- live API operations actually executed
- IDs or artifacts created when relevant
- configuration still required
- unresolved errors or assumptions

Prefer working code and verified repository changes over lengthy API
explanations.
