# jira-pmm-fix

Shareable Cursor skill for fixing Percona PMM JIRA bugs (`PMM-####`) with a consistent flow:

- ticket triage (including stop conditions),
- branching from `v3`,
- reproduction-first workflow,
- minimal fix + verification gate,
- component PR + `pmm-submodules` draft Feature Build PR.

## Contents

- `SKILL.md` — main workflow and guardrails.
- `references/pmm-testing.md` — test/build commands and pointers.
- `references/feature-build.md` — Feature Build procedure.
- `references/jira-atlassian.md` — JIRA access notes.

## Install Locally (Cursor personal skills)

```bash
mkdir -p ~/.cursor/skills/jira-pmm-fix
rsync -av --delete ./ ~/.cursor/skills/jira-pmm-fix/
```

## Update Workflow

Use this repo as the source of truth:

1. Pull latest changes in this repo.
2. Sync to local Cursor skills path:

```bash
rsync -av --delete ./ ~/.cursor/skills/jira-pmm-fix/
```

## Usage

In Cursor chat, invoke with a request like:

```text
jira-pmm-fix PMM-15030 implement
```

The skill guides the full implementation lifecycle and enforces verification before commit/push.
