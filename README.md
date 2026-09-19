# Adaption Agent Skills

Agent skills for integrating with the [Adaption](https://docs.adaptionlabs.ai/) API and SDK.

## Available skills

### `adaption`

Build, debug, review, and explain software integrations involving:

- Adaption datasets
- Adaptive Data
- dataset invention from scratch
- dataset preparation and augmentation
- AutoScientist
- experiment monitoring
- model and hyperparameter selection
- checkpoint downloads
- existing Adaption account state

The skill is designed to inspect the host repository first, follow existing project conventions, verify current API behavior against official Adaption documentation, and avoid unnecessary remote or billable operations.

## Structure

```text
skills/
└── adaption/
    └── SKILL.md
```

## Local installation

Copy the skill into a repository:

```text
<project>/
└── .agents/
    └── skills/
        └── adaption/
            └── SKILL.md
```

Or install it into your user-level agent skills directory:

```text
~/.agents/
└── skills/
    └── adaption/
        └── SKILL.md
```

## Usage

Invoke the skill from a supported coding agent:

```text
$adaption Review this project and add Adaption dataset ingestion.
```

```text
$adaption List my existing datasets and AutoScientist runs.
Do not modify anything.
```

```text
$adaption Diagnose why this AutoScientist integration is failing.
```

The skill checks current official Adaption documentation first and preserves conflicts between documentation, SDK behavior, UI behavior, and live API evidence instead of guessing.

## Authentication

Use:

```text
ADAPTION_API_KEY
```

Do not commit API keys to source control.

## Status

Initial validated release.
