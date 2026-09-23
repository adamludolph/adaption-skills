# Field notes: observed Adaption behavior

Operational observations collected while running a few hundred AutoScientist
experiments and Adaptive Data jobs in September 2026. Empirical observations
carry an evidence class from `SKILL.md` and the date they were seen; the final
verification section records working practices derived from that experience.
None of this is a documented contract. Re-verify anything here before it drives
automation, because deployed behavior changes. For SDK-specific observations
that do not identify a version, reproduce the behavior against the installed SDK
before treating it as current.

Contributed by Carson Rodrigues ([@rodriguescarson](https://github.com/rodriguescarson)).

## Run and dataset lifecycle

- **One active run per dataset, not one run per dataset.** *(verified, 2026-09-22)*
  Creating a second AutoScientist run on a dataset that already has an active
  run returns 409. Once that run is terminal, the same dataset accepts a new
  run; one dataset hosted four sequential runs. If you need parallel runs of the
  same rows, upload separate copies.
- **Observed practical concurrency around 5 active runs.** *(community observation, 2026-09-15 to 2026-09-22)*
  Launches beyond five active runs were refused or queued in these observations.
  Treat five as an observed operating point rather than a documented account
  limit, and have launchers wait or back off instead of retrying in a loop.
- **A run can report `succeeded` before its domain evaluation exists.**
  *(verified, 2026-09-22)* The held-out domain evaluation arrived 45 to 80
  minutes after the run and its job reported `succeeded`. An empty
  `domain_eval` right after completion may mean "not yet"; do not immediately
  classify it as lost. Poll with a deadline and only record a loss once the
  deadline passes.
- **`eval_failed` can repeat on one dataset while its neighbours pass.**
  *(community observation, 2026-09-22)* One dataset failed evaluation on all
  three iterations (training completed each time, error message null), while
  datasets launched alongside it evaluated normally. Its completions were mostly
  in non-Latin scripts (Kannada, Devanagari) and diacritic-heavy romanisation.
  *(hypothesis)* Script-heavy completions may break the judge or a parser. Not
  proven; treat it as the first variable to change, one at a time, as `SKILL.md`
  advises.

## Evaluation noise

- **Domain evaluation sample size varies per run.** *(verified, 2026-09-22)*
  Observed judged-prompt counts on the same day: 45, 47, 53, 54, 55, 60, 70, 73,
  74, 75, 76, 82, 99, 100. At n = 45 and a win rate near 0.87, the binomial
  standard error for one proportion is about 5 points; comparing two independent
  runs of similar size has still more uncertainty (about 7 points for the
  difference if both are n = 45 near that rate). Single-run differences of a few
  percentage points can therefore be sampling noise. Use replication or an
  uncertainty interval before attributing a difference to the experimental
  change.
- **Replicates of an identical configuration spread widely.** *(verified,
  2026-09-15 to 2026-09-22)* Repeat runs on unchanged rows and settings spread
  by more than ten domain points. Budget for replication before calling a
  difference, and never gate a controlled comparison on one draw.
- **Task win rate and domain score move separately.** *(verified, 2026-09-22)*
  A run with a 100% on-dataset win rate scored 84 on its domain evaluation, and
  runs with lower on-dataset rates scored higher. On-dataset win rate is a
  training-fit signal, not a forecast of held-out performance.

## Row floors and cost

- **Minimum training rows depend on the base model.** *(verified, 2026-09-16 to
  2026-09-21)* Every model refused 300 rows ("Fine-tuning requires at least 1,000
  training rows"). NVIDIA Nemotron 3 Super additionally refused a 6,000-row
  dataset at create time and needed at least 10,000. The count is rows after
  processing and deduplication, not rows in your file.
- **The fine-tune calculator does not know the per-model floor.** *(verified,
  2026-09-21)* `/finetune/calculate` returned `minTrainingRows: 1000` with
  `is_estimate_stub: true` for Nemotron. Trust the create-call error, not the
  estimate.
- **Invent charged on rows requested, not rows delivered.** *(verified,
  2026-09-16)* A 1,000-row request returned 588 rows and was charged in full.
  Record requested and delivered counts separately, as `SKILL.md` already asks.
- **Adaptive Data runs far slower than its estimate.** *(verified, 2026-09-21)*
  A job estimated at about 2.5 hours ran at 6 to 19 rows per minute, roughly 20
  hours for 11,000 rows. Credits were deducted at launch. Plan the downstream run
  around observed throughput, not the estimate.

## SDK and HTTP details

- **Job IDs live in `finetune_job_id`.** *(verified, 2026-09-22)* The list at
  `GET /api/v1/datasets/{dataset_id}/finetune/jobs` returns records keyed by
  `finetune_job_id`, not `id`. Passing a missing key through produced
  `/jobs/None/metrics` and a 500 that looked like a server fault.
- **The SDK's generic `get` needs the `/api/v1` prefix.** *(verified, 2026-09-22)*
  With `base_url = https://api.prod.adaptionlabs.ai`, `client.get("/datasets/...")`
  returned 404; `client.get("/api/v1/datasets/...", cast_to=object)` worked.
- **The metrics endpoint throttles hard.** *(verified, 2026-09-22)* Per-job
  metrics calls returned `429 ThrottlerException` whenever a background watcher
  was also polling, and exhausted the SDK's built-in retries. Poll every few
  minutes with a long backoff, and share one poller rather than running several.
- **Large uploads may need a longer write timeout.** *(community observation, 2026-09-17)* Files
  above about 10 MB timed out on the default `httpx` write timeout in the
  observed client/network environment. Some connections also stalled on IPv6;
  forcing IPv4 resolved it there. Treat both as client-side diagnostics, not
  platform requirements.
- **`upload_file` takes `column_mapping`, not arbitrary detection flags.**
  *(verified, 2026-09-22)* A `domain_detection=` keyword raised `TypeError`. Raw
  uploads need `processing_mode="raw"` plus an explicit prompt/completion
  `column_mapping`, then `wait_for_completion` before launch.
- **`data_format` is settable at run creation.** *(verified, 2026-09-22)* The
  create call accepts `data_format` of `chat` or `instruction`; raw datasets
  default to `chat`. On identical rows the two gave the same on-dataset win rate
  (99.2 each) and domain scores inside the evaluation noise above. No effect was
  measurable from one pair.
- **Fine-tunes ran on Together AI underneath.** *(verified, 2026-09-22)* Job
  records carried `"provider": "together_ai"`. Relevant if your organisation has
  provider restrictions; not something the create call lets you choose.

## Domain assignment

- **The evaluated domain is decided by the platform, not the request.**
  *(verified, 2026-09-16 to 2026-09-22)* Domain pinning in Invent did not control
  which domain a run was evaluated in, and the same rows were assigned different
  domains on different base models. A dataset's record exposed no domain field
  before training, and `domain_distribution_source` was sometimes null
  afterwards.
- **An external classifier is not a reliable proxy for that assignment.**
  *(verified, 2026-09-22)* A 60-prompt plurality vote by a separate LLM matched the
  platform's assignment in 2 of 5 topical datasets and 0 of 1 reasoning datasets.
  Check the assigned domain on the result before reporting a run as evidence
  for a particular domain.

## Hyperparameters

- **Hand-tuned adapters underperformed the platform-derived configuration in four comparisons.**
  *(community observation, 2026-09-16 to 2026-09-22)* Four comparisons on the same
  rows (larger LoRA rank, higher alpha, all-linear modules, two epochs) all scored
  below the platform-derived configuration, one by about 9 domain points. The
  best-launch-config endpoint showed that our strongest observed runs resolved
  to rank 8, alpha 8, one epoch, and learning rate 1e-4. Those values are
  observations from these runs, not fixed platform defaults. This supports the
  existing advice to prefer platform-derived hyperparameters.

## Verification habits that paid off

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
