# git-skill — Wildlife.ai
> Never assume branch state. Fetch first. Inspect. Act.

---

## PROHIBITED (no exceptions)

| ❌ Never | ✅ Instead |
|---|---|
| `git pull` | `git fetch origin --prune && git rebase origin/dev` |
| `git push --force` / `-f` | `git push --force-with-lease` |
| `git merge dev` on feature branch | `git rebase origin/dev` |
| `git fetch origin` without `--prune` | `git fetch origin --prune` |
| Commit to `dev` or `main` directly | Feature branch + PR always |
| Push without fetching first | Run PRE-PUSH below |
| Stacked PR base set to `dev` | Set base to parent feature branch |
| Push/commit with `CHANGES_REQUESTED` unresolved | Address comments first — run PR-REVIEW STATE check |
| Push/commit with unresolved inline threads | Resolve or dismiss threads — run PR-REVIEW STATE check |

---

## PRE-FLIGHT — every session, no exceptions

```bash
git fetch origin --prune
git status
git branch -vv
```

**`branch -vv` states:**

| Output | Action |
|---|---|
| `[origin/x]` | In sync — proceed |
| `[origin/x: ahead N]` | Push when ready |
| `[origin/x: behind N]` | Rebase before pushing |
| `[origin/x: ahead M, behind N]` | Rebase → `--force-with-lease` |
| `[origin/x: gone]` | `git branch -D <branch>` |

---

## BRANCH

```bash
git checkout dev
git fetch origin --prune && git rebase origin/dev
git branch -vv | grep dev           # must show in-sync before branching
git checkout -b <prefix>/<n>
```

**Prefixes:** `feature/` `fix/` `docs/` `chore/` `refactor/`  
**Stacked:** use sequence numbers — `feature/01-x`, `feature/02-y`, `feature/03-z`

---

## PRE-COMMIT CHECKLIST

Follow this checklist before every commit.

### 1. Read the Contributing Guidelines

Before any git operation, find and read the contributing guidelines. Check these locations **in order** and use the first one found:

1. **Repo-level**: `CONTRIBUTING.md` in the repo root, `.github/`, or `docs/`
2. **Organisation-level** (fallback): `https://github.com/wildlifeai/.github/blob/main/CONTRIBUTING.md`

Read and follow its branching strategy, commit conventions, and PR guidelines **exactly**.

> The contributing guidelines are the **single source of truth** for each repo's git workflow.
> If no repo-level file exists, the organisation default applies.

### 2. Review and Update Documentation

Before committing code changes, check if relevant documentation needs updating.

#### Where documentation lives

Search for `*.md` files in **all** of these directories (not just `docs/`):

| Priority | Path | Notes |
|----------|------|-------|
| 1 | `documentation/` | Primary docs directory (onboarding guides, resource guides) |
| 2 | `docs/` | Alternative docs directory (if present) |
| 3 | Repo root | `README.md`, `CHANGELOG.md`, etc. |

> **IMPORTANT**: The docs directory may vary per repo. Always check `documentation/` AND `docs/` — do not assume either.

Skip `node_modules/`, `archive/`, and similar non-documentation directories.

#### Steps

1. **Find documentation in the repo** — search for `*.md` files in the directories listed above that reference the changed functions, files, commands, or concepts.
2. **Read the relevant sections** — check if described behavior, command sequences, step tables, or architecture descriptions are now outdated by the code change.
3. **Update if needed** — update step tables, flow descriptions, architecture notes, and the "Last Updated" date.
4. **Skip if not needed** — minor refactors, formatting, test-only changes, or changes to undocumented code do not require doc updates.
5. **Include doc changes in the same commit** — documentation updates should ship with the code change, not as a separate commit.

---

## COMMIT

```bash
git add -A && git commit -m "<type>: <description>"
```

**Types:** `feat` `fix` `docs` `chore` `refactor` `test`  
**Body lines ≤ 100 chars** (commitlint CI fails otherwise):

```bash
git log -1 --format="%B" | awk 'length > 100 {print NR": "$0}'
# non-empty output → git commit --amend
```

---

## PRE-PUSH — every push, no exceptions

```bash
git fetch origin --prune
git branch -vv | grep "$(git branch --show-current)"
git rebase origin/dev
git log --oneline origin/dev..HEAD   # only YOUR commits must appear
git status                           # must be clean
```

Stop if `log` shows teammates' commits — rebase did not land cleanly.

If this branch has an open PR, also run **PR-REVIEW STATE** before pushing.

---

## PUSH

```bash
git push -u origin <branch>                      # first push
git push origin <branch>                         # subsequent
git push --force-with-lease origin <branch>      # after rebase/amend
```

---

## PULL REQUEST

```bash
git diff origin/dev...HEAD           # review full diff before opening
git log --oneline origin/dev..HEAD   # verify only intended commits
gh pr create --base dev --draft
```

- **base:** `dev` (or parent feature branch if stacked — see STACKED below)
- Description must include: what/why · how to test · screenshots (if UI) · linked issues
- Add human reviewer immediately — do not wait for Gemini

---

## PR-REVIEW STATE — run before every commit/push on a branch with an open PR

GitHub surfaces three independent layers of review feedback. Check all three.

### Layer 1 — Review decision (top-level)

```bash
gh pr view --json reviewDecision --jq '.reviewDecision'
```

| Value | Meaning | Action |
|---|---|---|
| `null` | No reviews submitted yet | Proceed |
| `REVIEW_REQUIRED` | Reviewer assigned, not yet reviewed | Proceed — await review |
| `APPROVED` | All reviewers approved | Proceed |
| `CHANGES_REQUESTED` | Reviewer explicitly requested changes | **Stop — address comments first** |

### Layer 2 — Unresolved inline threads (comments + suggestions)

`CHANGES_REQUESTED` only reflects the top-level review decision. Inline comment
threads and code suggestions remain unresolved independently. Check them explicitly:

```bash
# Count unresolved, non-outdated threads (if PR exists)
gh pr view --json reviewThreads --jq '[.reviewThreads[] | select(.isResolved == false and .isOutdated == false)] | length' 2>/dev/null || echo "No open PR — skipping thread check"
```

- Result `0` → all threads resolved, proceed
- Result `> 0` → **stop** — resolve or explicitly dismiss each thread on GitHub before committing

> Code suggestions from reviewers are unresolved threads. Committing over them
> without applying or dismissing them leaves the PR diff misleading.

### Layer 3 — CI checks

```bash
gh pr checks                              # human-readable status of all checks
# or for a machine-readable failure count:
gh pr view --json statusCheckRollup   --jq '[.statusCheckRollup[] | select(.conclusion == "FAILURE")] | length'
```

- Any `FAILURE` → **warn** — pushing more commits is fine, but fix the root cause
- `PENDING` → CI still running, push is safe if Layers 1 and 2 are clear

### Combined one-liner (quick check)

```bash
gh pr view --json reviewDecision,reviews,statusCheckRollup,reviewThreads --jq '
{
  decision:    .reviewDecision,
  changes_req: ([(.reviews // [])[] | select(.state=="CHANGES_REQUESTED")] | length),
  threads:     ([(.reviewThreads // [])[] | select(.isResolved==false and .isOutdated==false)] | length),
  ci_fail:     ([(.statusCheckRollup // [])[] | select(.conclusion=="FAILURE")] | length)
}' 2>/dev/null || echo "no open PR — skipping review check"
```

If `decision` is `CHANGES_REQUESTED`, `changes_req > 0`, or `threads > 0` → **stop**.  
If `ci_fail > 0` → **warn**, investigate before pushing.

---

## STACKED BRANCHES

```
dev
 └── feature/01-parent       ← PR #1 targets dev
       └── feature/02-child  ← PR #2 targets feature/01-parent
```

After PR #1 merges:

```bash
git checkout dev && git fetch origin --prune && git rebase origin/dev
git checkout feature/02-child
git rebase origin/dev
git log --oneline origin/dev..HEAD   # verify
git push --force-with-lease origin feature/02-child
# GitHub: edit PR #2 base → dev
```

Repeat per branch in the chain.

---

## CONFLICT RESOLUTION

```bash
git add <file> && git rebase --continue   # after resolving each file
git rebase --abort                        # bail out entirely
```

---

## CLEANUP — after every merge

```bash
git checkout dev && git fetch origin --prune && git rebase origin/dev
git branch -d <merged>              # use -D if squash-merged
GONE_BRANCHES=$(git branch -vv | grep ': gone]' | awk '{print ($1=="*" ? $2 : $1)}'); [ -n "$GONE_BRANCHES" ] && echo "$GONE_BRANCHES" | xargs git branch -D
```

---

## DIAGNOSTICS

```bash
git fetch origin --prune && git branch -vv          # full state
git log --oneline origin/dev..HEAD                  # your commits above dev
git log --oneline HEAD..origin/dev                  # remote commits you lack
git log --oneline --graph --decorate --all | head -40
git log --oneline --follow -- <file>                # file history
```

---

## AI AGENT RULES

1. Run PRE-FLIGHT before every session
2. Run PRE-PUSH before every push — no exceptions
3. Never guess branch state — always `fetch --prune` + `branch -vv` first
4. After rebase: verify with `git log --oneline origin/dev..HEAD`
5. Before any PR: run `git diff origin/dev...HEAD`
6. If `branch -vv` shows `ahead M, behind N` → stop, surface diagnostics, wait for confirmation
7. If `log --oneline origin/dev..HEAD` shows unexpected commits after rebase → stop, do not push
8. Prefer safety over speed — surface state, never take destructive action under ambiguity
9. Before committing or pushing on a branch with an open PR, run PR-REVIEW STATE (all three layers)
10. If Layer 1 returns `CHANGES_REQUESTED` → stop, report to developer, do not commit or push
11. If Layer 2 returns unresolved thread count `> 0` → stop, list thread count, wait for developer to resolve or dismiss on GitHub
12. If Layer 3 returns CI failures → warn, do not silently proceed

---

## DECISION TREE

```
New branch?     fetch --prune → rebase origin/dev → checkout -b
Commit?         add -A → commit → check body ≤100 chars
Push?           PRE-PUSH → push (-u / normal / --force-with-lease)
PR?             diff + log → gh pr create --base dev --draft
PR-REVIEW STATE?
                  Layer 1: gh pr view --json reviewDecision
                    CHANGES_REQUESTED → stop
                  Layer 2: unresolved thread count > 0 → stop
                  Layer 3: gh pr checks — any FAILURE → warn
Stacked PR?     base = parent branch (not dev) → rebase after parent merges
Cleanup?        fetch --prune → branch -d → batch-delete gone branches
Confused?       DIAGNOSTICS — never guess
```

---

## QUICK REFERENCE

```bash
# session start
git fetch origin --prune && git status && git branch -vv

# new branch
git checkout dev && git fetch origin --prune && git rebase origin/dev
git checkout -b feature/<n>

# commit + body check
git add -A && git commit -m "feat: what and why"
git log -1 --format="%B" | awk 'length > 100 {print NR": "$0}'

# pre-push (every time)
git fetch origin --prune
git branch -vv | grep "$(git branch --show-current)"
git rebase origin/dev
git log --oneline origin/dev..HEAD

# push
git push -u origin <branch>                      # first
git push origin <branch>                         # subsequent
git push --force-with-lease origin <branch>      # post-rebase

# pr-review state (if PR exists — run before every commit/push)
gh pr view --json reviewDecision,reviews,statusCheckRollup,reviewThreads --jq '{decision:.reviewDecision,changes_req:([(.reviews // [])[]|select(.state=="CHANGES_REQUESTED")]|length),threads:([(.reviewThreads // [])[]|select(.isResolved==false and .isOutdated==false)]|length),ci_fail:([(.statusCheckRollup // [])[]|select(.conclusion=="FAILURE")]|length)}' 2>/dev/null

# pr
git diff origin/dev...HEAD && gh pr create --base dev --draft

# cleanup
git checkout dev && git fetch origin --prune && git rebase origin/dev
git branch -d <branch>
GONE_BRANCHES=$(git branch -vv | grep ': gone]' | awk '{print ($1=="*" ? $2 : $1)}'); [ -n "$GONE_BRANCHES" ] && echo "$GONE_BRANCHES" | xargs git branch -D
```
