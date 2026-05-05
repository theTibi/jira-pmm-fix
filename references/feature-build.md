# Feature Build (PMM)

When a change spans submodules or needs full server/client images for QA/CI, Percona uses a **Feature Build** flow via [Percona-Lab/pmm-submodules](https://github.com/Percona-Lab/pmm-submodules).

Summary from [CONTRIBUTING.md — Feature Build](https://github.com/percona/pmm/blob/main/CONTRIBUTING.md):

1. Create a **draft** PR in `Percona-Lab/pmm-submodules` that pins your component branches/commits.
2. Describe the change; link **all** related GitHub PRs (pmm, grafana, exporters, etc.) in the FB PR body with checkboxes.
3. Move from Draft to Open only if you are actually changing pmm-submodules content (rare).
4. After related PRs merge, close the FB PR and delete the branch (default), or merge to pmm-submodules only when required (infra cases).

Full procedure: [pmm-submodules README (v3 branch)](https://github.com/Percona-Lab/pmm-submodules/blob/v3/README.md) — follow the section on creating a feature build.

**Skill reminder:** list every PR URL in the final fix summary so reviewers can trace the FB.

**Branch names in `ci.yml`:** use the same remote branch string as in Git (prefer **`fix-pmm-<id>`** without `/`) so submodule automation and `ci.py` do not mis-parse hierarchical names.

---

## Checklist: component PR → pmm-submodules draft PR

Use the **same branch name** on the component repo(s) and on `pmm-submodules` (e.g. `fix-pmm-14894`).

1. **Open the component PR first** (e.g. `percona/pmm` → base `v3`, head `fix-pmm-<id>`). Copy the PR URL.
2. **Clone or update** `Percona-Lab/pmm-submodules`, then `git fetch origin && git switch -c fix-pmm-<id> origin/v3` (or rebase your existing FB branch).
3. **Edit `ci.yml`** at the repo root (create if missing). Minimum for a **`percona/pmm`-only** FB:

   ```yaml
   deps:
     - name: pmm
       branch: fix-pmm-<id>
       url: https://github.com/percona/pmm
   ```

   `branch` must be the branch you **just pushed** on that `url` repo. Add more list items if you also pin grafana, exporters, etc.

4. **Update submodule pointers** in the index (and any other files the README requires) when the process demands it so Jenkins builds the intended SHAs.
5. **`git add ci.yml`** (and submodule paths if changed), **commit**, **`git push -u origin fix-pmm-<id>`**.
6. **Open a draft PR** in `Percona-Lab/pmm-submodules`: base **`v3`**, head **`fix-pmm-<id>`**, title e.g. `fix-pmm-<id> (FB)`, body with checkboxes linking each **component PR URL**.
7. **Edit the component PR** to add the **pmm-submodules PR** link under Test plan / Feature Build.
