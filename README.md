# Adaption Agent Skill

A reusable Agent Skill for building, debugging, reviewing, and explaining integrations with the [Adaption](https://docs.adaptionlabs.ai/) API and Python SDK.

Works with **Codex**, **Claude Code**, and other agents that support the open Agent Skills format.

## Install

Install with the open `skills` CLI:

```bash
npx skills add adamludolph/adaption-skills
```

To install the `adaption` skill globally for both Codex and Claude Code:

```bash
npx skills add adamludolph/adaption-skills \
  --skill adaption \
  --global \
  --agent codex \
  --agent claude-code
```

You can also inspect the repository's available skills before installing:

```bash
npx skills add adamludolph/adaption-skills --list
```

## What it covers

The `adaption` skill supports work involving:

- Adaption datasets
- Adaptive Data
- dataset invention from scratch
- dataset preparation and augmentation
- raw/training-ready dataset ingestion
- AutoScientist
- experiment monitoring and diagnostics
- model and hyperparameter selection
- checkpoint downloads
- existing Adaption account state
- API, SDK, UI, and observed-behavior discrepancies
- experiment provenance and evidence handling

`skills/adaption/references/field-notes.md` preserves attributed September 2026 reports: evaluation timing and noise, per-model row floors, SDK and HTTP quirks, and hyperparameter comparisons. These reports are not independently reproduced by this package's documentation check. The notes also hold optional experiment guidance, unresolved documentation conflicts, and qualified October 2026 Invent research context.

The loaded skill focuses on Adaption-specific workflow decisions and consequential API behavior. Ordinary engineering, authority, and verification instructions belong to the host agent's operating contract and task instructions.

## Usage

### Codex

Invoke explicitly with:

```text
$adaption Review this project and add Adaption dataset ingestion.
```

```text
$adaption List my existing datasets and AutoScientist runs. Do not modify anything.
```

```text
$adaption Diagnose why this AutoScientist integration is failing.
```

Codex can also select the skill automatically when the task matches its description.

### Claude Code

Invoke explicitly with:

```text
/adaption Review this project and add Adaption dataset ingestion.
```

```text
/adaption Diagnose why this AutoScientist integration is failing.
```

Claude Code can also discover and use the skill automatically when the task matches its description.

## Manual installation

The canonical skill lives at:

```text
skills/adaption/SKILL.md
```

For a project-scoped install, copy the `adaption` directory to:

```text
Codex:       <project>/.agents/skills/adaption/
Claude Code: <project>/.claude/skills/adaption/
```

For a user-level install:

```text
Codex:       ~/.agents/skills/adaption/
Claude Code: ~/.claude/skills/adaption/
```

This GitHub repository is the authoring source of truth. Maintain `skills/adaption/SKILL.md` and its supporting references here; installed and loose local copies are deployment copies or historical inputs. Propose changes here before updating an installation.

Copy the complete `skills/adaption/` directory, including references, from a reviewed revision. Record that source revision and the installed file hashes when deploying so drift can be compared deliberately. A project-specific installation should not become an independently maintained fork without a demonstrated project requirement.

## Authentication

Set the Adaption API key using:

```text
ADAPTION_API_KEY
```

Do not commit API keys or other secrets to source control.

## Structure

```text
adaption-skills/
├── LICENSE
├── README.md
└── skills/
    └── adaption/
        ├── SKILL.md
        └── references/
            └── field-notes.md
```

## Design principles

The skill keeps these evidence classes distinct:

- documented contract
- reproducible/versioned observation
- attributed community observation
- hypothesis

It preserves conflicts between documentation, SDK behavior, UI behavior, and live API evidence instead of guessing, and it separates read-only discovery from authorization for paid or mutating operations.

## License

MIT
