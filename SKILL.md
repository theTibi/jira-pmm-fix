---
name: jira-pmm-fix
description: >
  End-to-end workflow for Percona PMM bugs from JIRA (PMM-####): read ticket, map to
  managed/agent/admin/api/ui/dashboards (or external Grafana fork), branch, reproduce
  with Go tests, api-tests, or UI tests, minimal fix, verify, optional external diff
  review, PR with ticket ID, Feature Build via pmm-submodules when required. Use for
  perconadev.atlassian.net PMM tickets, fixing PMM server/client/UI, api-tests,
  make env-up, reproducing PMM issues, or branching from origin/v3.
  Stops and asks the user when a ticket looks obsolete, invalid, or not worth a code fix.
  Uses slash-free branch names (e.g. fix-pmm-1234) for automation compatibility.
  Commit and PR titles use conventional style: fix: short description.
  Ends with component PR plus draft pmm-submodules Feature Build PR unless user skips FB.
disable-model-invocation: true
---

# JIRA → PMM → Fix

Linear workflow. Do **not** skip the **reproduction gate**: no agreed implementation plan until there is an automated repro, a documented manual repro the user accepts, or an explicit stop with repro gap.

For command details, see [references/pmm-testing.md](references/pmm-testing.md), [references/feature-build.md](references/feature-build.md), [references/jira-atlassian.md](references/jira-atlassian.md).

---

## 1. Read JIRA

- Issue key **PMM-####** on [Percona JIRA](https://perconadev.atlassian.net).
- Use Atlassian MCP / API if available; else ask the user to paste summary, description, steps, versions, components, attachments, and relevant comments.
- Capture: **Affects version**, Server vs Client, environment (K8s, Docker, OS), linked PRs/issues.

If the ticket cannot be read, stop.

### Triaging relevance (mandatory before branching)

After reading the ticket and doing a **light** sanity check (description, comments, fix versions, linked PRs, current `v3` behaviour if cheap to verify), **stop and tell the user**—do **not** create a branch, edit code, or plan an implementation—when you conclude any of the following:

- **Obsolete / no longer relevant** — already fixed on `v3`, duplicate of another ticket, feature removed, or only affected EOL versions with no supported backport path.
- **Not a bug** — works as designed, user misunderstanding, needs documentation or training instead of code.
- **Not a sensible PMM code change** — wrong product, support/config issue only, or no credible change in this repo that matches the report.
- **Should not be fixed** — security, compatibility, or product policy implies **Won’t fix** / **Not a bug** is the right JIRA outcome.
- **Stale or empty** — insufficient detail to act, and guessing would be irresponsible; ask for clarification instead of a silent dry-run.

When stopping, be **explicit**: quote what you checked, why the ticket should not drive a fix (or should be handled differently), and what you recommend (e.g. close as Duplicate, resolve as Won’t Fix, open doc ticket, ask reporter for version + repro). **Wait for the user** if they might still want the work (override, different scope, or “implement anyway for learning”).

Only proceed to **§2 Create branch** when the ticket **passes this triage** or the user **explicitly overrides** after your stop.

---

## 2. Create branch

From the correct repository (often `percona/pmm`; sometimes `percona/grafana` or others).

**Default base branch is `v3`** for Percona PMM work (upstream `origin/v3`). Only use another base (e.g. `main`, a feature branch) if the user explicitly says so.

**Branch naming (default):** use **`fix-pmm-<id>`** — lowercase, **hyphens only**, no **`/`** in the branch name (e.g. ticket **PMM-14894** → branch **`fix-pmm-14894`**). Slashes in names like `fix/pmm-14894` break some automation (e.g. scripts that parse `git ls-remote` or Jenkins paths). Use the **same name** across repos for a Feature Build (`ci.yml` `branch:` must match the real remote branch).

```bash
git fetch origin && git switch -c fix-pmm-<id> origin/v3
```

If a fix branch already exists on the wrong base, **rebase or recreate it onto `v3`** (e.g. `git reset --hard origin/v3` then `git cherry-pick <fix-commit>`) so the PR targets the correct line.

Use a different naming scheme only if the user or team docs require it. Create the branch **before** editing tracked files.

---

## 3. Route to code

| Signal | Primary area | Typical paths |
|--------|----------------|---------------|
| Server config, VictoriaMetrics, supervisord, PMM APIs orchestration | managed | `managed/` |
| Scraping, exporters, vmagent, node agent | agent | `agent/` |
| CLI (`pmm-admin`, etc.) | admin | `admin/` |
| Protos, OpenAPI, generated API surface | api | `api/` |
| PMM standalone UI (not Grafana shell) | ui | `ui/` |
| Dashboard JSON / PMM app dashboards | dashboards | `dashboards/` |
| Inventory/install UI inside Grafana fork | grafana fork | separate clone (e.g. `public/app/percona/...`) |

If multiple areas: list each and note whether a **Feature Build** is required (see [references/feature-build.md](references/feature-build.md)).

---

## 4. Reproduce

Pick the **lowest** layer that still proves the bug:

1. **Go unit / integration** in the affected module (`go test ./path/...`).
2. **api-tests** — requires PMM Server running; see [references/pmm-testing.md](references/pmm-testing.md).
3. **UI** — Vitest via `ui` workspace; scoped package when possible.
4. **E2E** — separate repos (e.g. pmm-qa); only if lower layers cannot cover the failure mode.

If reproduction needs conditions you cannot automate in-repo, document a **minimal manual checklist**, get user acknowledgment, and **stop** the “implement fix” path until repro is agreed or scope changes.

### Bug not reproduced

If tests pass unexpectedly: propose hypotheses (wrong version, wrong component, missing flag/service, stale cache). Propose **one** change at a time. **Wait for user confirmation** before retrying.

---

## 5. Local environment (when needed)

- Server stack: from PMM repo root, `make env-up` (devcontainer). First run can be slow; `make env-up-rebuild` when the image must refresh.
- Iterating **pmm-managed** inside the container: follow current repo docs (e.g. `make run-managed` / `Makefile.devcontainer` targets — verify in-tree before running).
- Agent: under `agent/`, `make setup-dev`, then `make run`; see agent README.
- UI: under `ui/`, `make setup`, `make dev` or `yarn test` via turbo as documented.

Always prefer **Makefile and README in the checkout** over memorized flags.

---

## 6. Plan the fix (after repro)

- Trace the failing path (handler → service → DB, or UI data flow).
- State root cause in plain language.
- Minimal change, risks, rollback, and **which tests** will prove it.
- If `api/` protos or generated code change: include **regeneration** steps from repo docs.

For ambiguous product behavior, wait for explicit user or ticket agreement before implementing.

---

## 7. Implement

- Smallest diff; match existing style and error handling.
- Do not widen scope (no drive-by refactors).
- Regenerate API/OpenAPI artifacts when required; commit generated outputs if the repo expects it.
- Next step is always **§9 Verify** before any **§10** commit or push (see §9 for the “no push without a verification step” rule).

---

## 8. Optional external review

Pipe `git diff` into your team’s preferred reviewer (CLI or another model). Classify findings; address non-trivial issues before merge.

---

## 9. Verify

- Confirm the repro test **failed before** and **passes after** (or equivalent evidence).
- Run broader tests for touched packages when risk warrants it.
- **Do not commit or push on code inspection alone**, even when the bug and a one-line fix look obvious from reading the tree. After implementing, **always** run at least one verification step that touches the changed behaviour: targeted `go test`, `api-tests` with a running server (see [references/pmm-testing.md](references/pmm-testing.md)), UI `yarn test` / build for touched packages, `nginx -t` / image smoke if nginx or packaging changed, or an **agreed minimal manual checklist** (e.g. curl against a local or FB server) with outcomes noted in the PR test plan. Only **then** proceed to **§10**.
- If verification cannot be run in this environment (no Docker, no external VM, etc.), **stop before commit**: report what was run vs blocked, and ask the user to run the missing step or waive in writing—do not push untested changes by default.
- Default workflow: after the **component** PR exists, open a **draft** [pmm-submodules](https://github.com/Percona-Lab/pmm-submodules) PR unless the user said **no FB** (e.g. docs-only).

---

## 10. Commit, push, and open PRs

**Order:** complete **§9 Verify** first; **§10** is only after there is concrete evidence (command output, test names, or user-accepted manual results).

**Commit and PR title (default):** use **[Conventional Commits](https://www.conventionalcommits.org/)** style — **`fix: <imperative short description>`** (lowercase `fix`, colon, space, then what changed). Example: `fix: show valkey cluster messages as rate (PMM-14894)`. Put extra context, root cause, and ticket link in the **body** if needed. Omit `(<area>)` scopes unless the repo already standardizes on them (e.g. `fix(ui):`); prefer a clear **`fix:`** subject for consistency.

### 10.1 Component repository (e.g. `percona/pmm`, `percona/grafana`)

```bash
git add <paths>
git status --short
git commit -m "fix: <imperative short description> (PMM-<id>)

<optional body: root cause, what changed>"
git push -u origin fix-pmm-<id>
```

Open the **component** pull request against the agreed base (**`v3`** for `percona/pmm` unless the user said otherwise). Use the **same first line** as the commit for the PR title, e.g.:

```bash
gh pr create --repo percona/pmm --base v3 --head fix-pmm-<id> \
  --title "fix: ... (PMM-<id>)" \
  --body-file /tmp/pr-body.md
```

If `gh` is unavailable, use the compare URL GitHub shows after push.

**Component PR body** should include a placeholder or link for the Feature Build once it exists:

```markdown
## Summary
- **Bug**: ...
- **Root cause**: ...
- **Fix**: ...

## Ticket
- PMM-<id>

## Test plan
- [ ] ...
- [ ] **pmm-submodules (FB) PR:** <add link after step 10.2>
```

### 10.2 `Percona-Lab/pmm-submodules` (draft Feature Build PR)

**Default:** after the component PR branch is pushed, also open a **draft** PR in **[Percona-Lab/pmm-submodules](https://github.com/Percona-Lab/pmm-submodules)** targeting **`v3`**, using the **same branch name** as the component work (**`fix-pmm-<id>`**, no `/`). Skip this step only when the user explicitly opts out (documentation-only, submodule-only infra, etc.).

More detail: [references/feature-build.md](references/feature-build.md).

1. In your `pmm-submodules` clone: `git fetch origin && git switch -c fix-pmm-<id> origin/v3` (or rebase an existing FB branch).
2. **Edit `ci.yml` at the repo root** (create the file if it does not exist). It must list the **`pmm` repo and the branch you just pushed** so `ci.py` / Jenkins know where to pull from. Shape:

   ```yaml
   deps:
     - name: pmm
       branch: fix-pmm-<id>
       url: https://github.com/percona/pmm
   ```

   For work that also pins **grafana**, exporters, etc., add more `- name: …` / `branch:` / `url:` entries—each `branch` must be the real remote branch name (same `fix-pmm-<id>` pattern unless you use another agreed name).

3. **Submodule gitlinks:** if the [pmm-submodules v3 README](https://github.com/Percona-Lab/pmm-submodules/blob/v3/README.md) requires bumping `sources/pmm/...` (or other submodules) to the commit Jenkins should build, do that in the same commit or a follow-up per team process.

4. **Commit and push** on `fix-pmm-<id>`:

   ```bash
   git add ci.yml
   git status --short   # add submodule paths if step 3 changed them
   git commit -m "chore: feature build for PMM-<id>"
   git push -u origin fix-pmm-<id>
   ```

5. **Open the draft PR** (base **`v3`**, head **`fix-pmm-<id>`**):

   ```bash
   gh pr create --repo Percona-Lab/pmm-submodules --base v3 --head fix-pmm-<id> --draft \
     --title "fix-pmm-<id> (FB)" \
     --body "## Feature build for PMM-<id>
   - [ ] percona/pmm: <component PR URL>
   "
   ```

6. **Cross-link:** put the **pmm-submodules PR URL** into the component PR body (Test plan / FB line) and keep the component PR link in the FB body.

---

## Completion summary

```markdown
## Fix summary for PMM-<id>

**Bug**:
**Root cause**:
**Fix**:
**Tests**:
**Component PR(s)**:
**pmm-submodules (FB) PR**:
```
