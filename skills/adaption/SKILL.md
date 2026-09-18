---
name: adaption
description: >
  Build, debug, review, and explain software integrations with the
  Adaption API and Python SDK. Use this skill for Adaption datasets,
  Adaptive Data, dataset preparation and augmentation, evaluation,
  AutoScientist model training, training experiments, model selection,
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
- Never invent API methods, request fields, response fields, status
  values, model identifiers, or training modes.

## Determine the task

Classify the request as one or more of:

- explain an API or workflow
- review existing integration code
- debug an integration
- create/import/upload a dataset
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

## Long-running operations

Treat dataset processing, adaptation, and training as asynchronous when
documented.

A robust integration should:

1. store returned dataset/run/experiment IDs
2. monitor status
3. recognize successful terminal states
4. recognize failure/cancellation states
5. expose useful error details
6. support reasonable timeout behavior
7. avoid duplicate creation when execution is retried
8. support resume behavior when appropriate for the host application

Use documented idempotency support for retryable creation operations
where available.

## Cost and remote execution

Generating integration code does not automatically authorize unnecessary
paid API operations.

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
