# Field notes: attributed Adaption observations

The contributor reports experience from a few hundred AutoScientist experiments
and Adaptive Data jobs in September 2026. The September reports originated in
[commit 0324353](https://github.com/adamludolph/adaption-skills/commit/03243530388ce998eaed995bb1f812e32f590d19).
The supplied notes do not attach run IDs, raw request/response artifacts, or SDK
versions sufficient to independently reproduce the reports. Previously labelled
"verified" entries are therefore retained as **attributed community observations**,
with their reported dates. They do not establish current platform contracts.

Reproduce a relevant report against the actual endpoint/SDK before relying on it
for automation. Any future reproducible/versioned observation should include
the request, response/error, dataset/run/iteration IDs, timestamp, environment,
SDK version, and reproduction procedure. Hypotheses remain separate from reports.

Contents: documentation conflicts; Invent research; lifecycle; evaluation noise; row floors/cost;
SDK/HTTP; domain assignment; hyperparameters; optional experiment practices.

Contributed by Carson Rodrigues ([@rodriguescarson](https://github.com/rodriguescarson)).

## Current documentation checks and open conflicts

Checked **2026-10-02**, through public documentation only; no live SDK or paid
operation was performed. Documented workflows are linked from [SKILL.md](../SKILL.md).

- Both the [running guide](https://docs.adaptionlabs.ai/autoscientist/running-autoscientist/)
  and [create reference](https://docs.adaptionlabs.ai/api/resources/autoscientist/methods/create)
  now specify 1–5 iterations. Historical references to 1–10 are not current guidance.
- The create reference describes the target interval as inclusive, but also renders
  `exclusiveMinimum`/`exclusiveMaximum` flags. The running guide says greater than
  0.5 and at most 1. Boundary acceptance remains **unresolved**; check the installed
  SDK schema and supported endpoint before relying on a boundary value.
- The [Invent guide](https://docs.adaptionlabs.ai/adaptive-data/invent-a-dataset/)
  uses `succeeded` for completion; the
  [dataset download reference](https://docs.adaptionlabs.ai/api/resources/datasets/methods/download)
  describes full output for `ready` and partial output for `failed`. Preserve the
  resource/SDK-specific labels; a single universal dataset status enum has not been
  established here. The partial-download behavior does not authorize training on
  an incompletely processed dataset.
- The [create reference](https://docs.adaptionlabs.ai/api/resources/datasets/methods/create)
  accepts raw prompt/chosen/rejected mappings, while the
  [raw guide](https://docs.adaptionlabs.ai/autoscientist/run-on-non-adapted-data/)
  says preference data requires adaptation. Raw pair ingestion is documented;
  its alignment eligibility must be checked through the processed dataset type
  and resolved run method. Do not transform existing pairs merely to follow the
  older guide. No live raw-pair/alignment test was performed here.
- The [adapt reference](https://docs.adaptionlabs.ai/api/python/resources/datasets/methods/adapt)
  describes adaptation as preserving counts, yet also supports language expansion
  and deduplication. Count preservation is not a blanket guarantee. Its mapping
  prose requires context for a universal prompt but separately allows an image
  differentiator; validate the actual request schema when relying on image alone
  alongside shared text framing. The reference describes length generally; the
  [configuration guide](https://docs.adaptionlabs.ai/adaptive-data/configure-adaptive-data/)
  ties it to prompt rephrasing. Inspect effective recipes when behavior differs.

## Invent research scope and results

**Research, not API contract.** The October 1, 2026
[technical report](https://www.alphaxiv.org/pdf/2609.invent-a-dataset-zero-seed)
was inspected in its primary PDF viewer (pp. 1, 3, 7–12, 14) on 2026-10-02.
The [official blog](https://adaptionlabs.ai/blog/measuring-dataset-generation-abilities-with-zero-seed-data)
and Discord announcement are leads; neither substitutes for API documentation.

The report targets generation from natural-language specifications without seed
rows. Its scope is text instruction/preference data for SFT/alignment; limitations
exclude tool-call traces, multi-step agent trajectories and non-text modalities.
Aggregate tested quality/diversity results favor Invent, but the separate relevance
metric does not. Downstream medical-QA comparisons are architecture-dependent:
Invent fine-tunes lead for Llama/Gemma, while the untuned Qwen base leads its
Invent fine-tune. These judge-based, task/model-specific results do not establish
universal gains or support untested workflow/modality claims. Headline percentages
are deliberately omitted from operational guidance.

## Run and dataset lifecycle

- **One active run per dataset, not one run per dataset.** *(community observation, 2026-09-22)*
  Creating a second AutoScientist run on a dataset that already has an active
  run returns 409. Once that run is terminal, the same dataset accepts a new
  run; one dataset reportedly hosted four sequential runs. Separate uploads were
  used to obtain parallel runs; this is a reported workaround, not an instruction
  to duplicate datasets or launch additional paid work on a 409 response.
- **Observed practical concurrency around 5 active runs.** *(community observation, 2026-09-15 to 2026-09-22)*
  Launches beyond five active runs were refused or queued in these observations.
  Treat five as an observed operating point rather than a documented account
  limit, and have launchers wait or back off instead of retrying in a loop.
- **A run can report `succeeded` before its domain evaluation exists.**
  *(community observation, 2026-09-22)* The held-out domain evaluation arrived 45 to 80
  minutes after the run and its job reported `succeeded`. An empty
  `domain_eval` right after completion may mean "not yet"; do not immediately
  classify it as lost. Poll with a deadline; if the result remains missing at
  that deadline, record it as unresolved, not as a measured loss or zero score.
- **`eval_failed` can repeat on one dataset while its neighbours pass.**
  *(community observation, 2026-09-22)* One dataset failed evaluation on all
  three iterations (training completed each time, error message null), while
  datasets launched alongside it evaluated normally. Its completions were mostly
  in non-Latin scripts (Kannada, Devanagari) and diacritic-heavy romanisation.
  *(hypothesis)* Script-heavy completions may break the judge or a parser. Not
  proven; the report does not establish that script choice should be the first
  variable changed. Compare format, lengths, mappings, and other observable
  differences, then choose a test supported by the actual failure evidence.

## Evaluation noise

- **Domain evaluation sample size varies per run.** *(community observation, 2026-09-22)*
  Observed judged-prompt counts on the same day: 45, 47, 53, 54, 55, 60, 70, 73,
  74, 75, 76, 82, 99, 100. If outcomes were independent binary judgements, at
  n = 45 and a proportion near 0.87 the binomial standard error would be about
  5 percentage points; for two independent runs at that size and rate, about
  7 points for their difference. The report does not establish those assumptions
  for the platform's scoring process. Single-run differences of a few
  percentage points can therefore be sampling noise. Use replication or an
  uncertainty interval before attributing a difference to the experimental
  change.
- **Replicates of an identical configuration spread widely.** *(community observation,
  2026-09-15 to 2026-09-22)* Repeat runs on unchanged rows and settings spread
  by more than ten domain points. Budget for replication before calling a
  difference, when the experiment's budget permits. This report does not estimate
  a universal variance or establish that every evaluation uses that sampling process.
- **Task win rate and domain score move separately.** *(community observation, 2026-09-22)*
  A reported run with a 100% on-dataset win rate scored 84 on its domain evaluation,
  and runs with lower on-dataset rates scored higher. The example motivates
  separating the two metrics; it does not calibrate held-out performance.

## Row floors and cost

- **Minimum training rows depend on the base model.** *(community observation, 2026-09-16 to
  2026-09-21)* The models tested reportedly refused 300 rows ("Fine-tuning requires at least 1,000
  training rows"). NVIDIA Nemotron 3 Super additionally refused a 6,000-row
  dataset at create time and needed at least 10,000. The count is rows after
  processing and deduplication in these reports, not rows in the source file.
  Recheck current requirements for the selected model and objective; these
  historical values are not maintained as universal floors.
- **The fine-tune calculator does not know the per-model floor.** *(community observation,
  2026-09-21)* `/finetune/calculate` returned `minTrainingRows: 1000` with
  `is_estimate_stub: true` for Nemotron. Trust the create-call error, not the
  estimate. This historical internal endpoint is not a substitute for the current
  supported model/launch requirements.
- **Invent charged on rows requested, not rows delivered.** *(community observation,
  2026-09-16)* A 1,000-row request returned 588 rows and was charged in full.
  Record requested and delivered counts separately. No invoice or account policy
  is attached; this report does not establish current universal billing semantics.
- **Adaptive Data runs far slower than its estimate.** *(community observation, 2026-09-21)*
  A job estimated at about 2.5 hours ran at 6 to 19 rows per minute, roughly 20
  hours for 11,000 rows. Credits were deducted at launch. Plan the downstream run
  around observed throughput, not the estimate.

## SDK and HTTP details

- **Job IDs live in `finetune_job_id`.** *(community observation, 2026-09-22)* The list at
  `GET /api/v1/datasets/{dataset_id}/finetune/jobs` returns records keyed by
  `finetune_job_id`, not `id`. Passing a missing key through produced
  `/jobs/None/metrics` and a 500 that looked like a server fault.
- **The SDK's generic `get` needs the `/api/v1` prefix.** *(community observation, 2026-09-22)*
  With `base_url = https://api.prod.adaptionlabs.ai`, `client.get("/datasets/...")`
  returned 404; `client.get("/api/v1/datasets/...", cast_to=object)` worked.
- **The metrics endpoint throttles hard.** *(community observation, 2026-09-22)* Per-job
  metrics calls returned `429 ThrottlerException` whenever a background watcher
  was also polling, and exhausted the SDK's built-in retries. Poll with backoff
  and coordinate duplicate pollers when that reproduces. A fixed
  polling interval or platform-wide throttle limit is not established here.
- **Large uploads may need a longer write timeout.** *(community observation, 2026-09-17)* Files
  above about 10 MB timed out on the default `httpx` write timeout in the
  observed client/network environment. Some connections also stalled on IPv6;
  forcing IPv4 resolved it there. Treat both as client-side diagnostics, not
  platform requirements.
- **`upload_file` takes `column_mapping`, not arbitrary detection flags.**
  *(community observation, 2026-09-22)* A `domain_detection=` keyword raised `TypeError`. These raw
  uploads used `processing_mode="raw"` plus an explicit prompt/completion
  `column_mapping`, then `wait_for_completion` before launch.
- **`data_format` is settable at run creation.** *(community observation, 2026-09-22)* The
  create call accepts `data_format` of `chat` or `instruction`; raw datasets
  default to `chat`. On identical rows the two gave the same on-dataset win rate
  (99.2 each) and domain scores inside the evaluation noise above. No effect was
  measurable from one pair. That comparison does not establish equivalence.
  Current documented encoding behavior is described in [SKILL.md](../SKILL.md); the
  experiment result remains an attributed observation.
- **Fine-tunes ran on Together AI underneath.** *(community observation, 2026-09-22)* Job
  records carried `"provider": "together_ai"`. Relevant if your organisation has
  provider restrictions. This record does not establish the current processor
  for every job, selectable provider options, or a privacy contract; use current
  provider documentation or account-specific confirmation for those decisions.

## Domain assignment

- **The evaluated domain is decided by the platform, not the request.**
  *(community observation, 2026-09-16 to 2026-09-22)* Domain pinning in Invent did not control
  which domain a run was evaluated in, and the same rows were assigned different
  domains on different base models. A dataset's record exposed no domain field
  before training, and `domain_distribution_source` was sometimes null
  afterwards. This is a report about those runs, not a universal domain-assignment
  rule or a promise about current fields.
- **An external classifier is not a reliable proxy for that assignment.**
  *(community observation, 2026-09-22)* A 60-prompt plurality vote by a separate LLM matched the
  platform's assignment in 2 of 5 topical datasets and 0 of 1 reasoning datasets.
  Check the assigned domain on the result before reporting a run as evidence
  for a particular domain. These small samples do not establish a general
  classifier accuracy or replacement evaluation method.

## Hyperparameters

- **Hand-tuned adapters underperformed the platform-derived configuration in four comparisons.**
  *(community observation, 2026-09-16 to 2026-09-22)* Four comparisons on the same
  rows (larger LoRA rank, higher alpha, all-linear modules, two epochs) all scored
  below the platform-derived configuration, one by about 9 domain points. The
  best-launch-config endpoint showed that our strongest observed runs resolved
  to rank 8, alpha 8, one epoch, and learning rate 1e-4. Those values are
  observations from these runs, not fixed platform defaults. This supports the
  use of the supported recommendation operation rather than frozen constants;
  the comparisons do not prove derived recipes outperform every override.

## Optional experiment practices

These are methods for Adaption experiment work, not provider requirements or
independently validated findings. Apply them when the task actually compares runs.

- Preserve the submitted request and resolved settings, including failed,
  cancelled, and incomplete runs. Prefer paired comparisons on the same dataset
  or seed, isolate material changes, keep search budgets comparable, and replicate
  important results when practical. Raising the stopping threshold changes the
  permitted search; no fixed threshold such as `0.95` is a universal recipe.
- Keep quality evaluation, application classification, training win rate,
  held-out/domain scores, leaderboard normalization, and local metrics distinct.
  For a held-out result, record whether it ran, its evaluated iteration, timestamp,
  assigned domain, population, and judged count when observable. Missing or
  nondeterministically assigned evaluations cannot alone settle a controlled
  comparison; retain their uncertainty rather than converting it into a score.

- **Pre-register the pass mark before launch.** Write down the score that counts
  as a success, a failure, and "inside the noise" before the run exists. It
  stopped several borderline results from being argued into wins.
- **A checker must not share the generator's assumptions.** When a dataset is
  generated programmatically, verify it with a separately written check that
  re-derives each label from the stored prompt text. Then plant known defects and
  confirm the checker catches all of them; a checker that reports zero failures
  without that control tells you nothing.
- **Read what was written, not what was promised.** Compare downloaded byte
  counts with the declared length, re-count rows after upload, and confirm the
  run's recorded dataset ID matches the one you meant to launch.
