# Contributing to Wildlife.ai 🌿

Kia ora! To keep our codebases healthy across all projects, we follow a strict **linear history** and **review-first** workflow.

## ⚖️ Core Principles

1.  **Linear History:** We rebase; we don't merge bubbles.
2.  **Review Everything:** No code lands in `dev` without a second pair of eyes (Gemini + Human).
3.  **Default to `dev`:** `main` is for stable releases only.

---

## 🌿 Branching Strategy

| Branch | Purpose | Direct commits? |
| :--- | :--- | :--- |
| `main` | Stable production releases | **Never** — only merged from `dev` via PR |
| `dev` | Integration branch (head branch for all work) | **Never** — only merged from feature branches via PR |
| `feature/*` | Individual task, fix, or experiment | **Yes** |

> [!IMPORTANT]
> All work starts and ends with `dev`. Never commit directly to `dev` or `main`.

---

## 🚀 Standard Workflow

### 1. Start a New Task

Always branch from the latest `dev` to ensure you are building on the most recent work:

```bash
# Make sure you are on dev and it is up to date
git checkout dev
git pull origin dev

# Create a new branch for your task
git checkout -b feature/your-descriptive-name
```

Use a short, descriptive name: `feature/ble-timeout-fix`, `feature/add-lora-retry`, etc.

---

### 2. Do Your Work — Commit Often

Make small, focused commits as you work. We use [Conventional Commits](https://www.conventionalcommits.org/) (e.g., `feat:`, `fix:`, `docs:`) to help automate release notes.

```bash
git add -A
git commit -m "fix: add retry logic for BLE command timeout"
```

Use clear commit messages that explain **what** and **why**.

---

### 3. Before Pushing — Rebase onto `dev`

Before you push (or create a PR), rebase your branch onto the latest `dev` to keep a linear history. This avoids messy "merge bubbles."

```bash
# Fetch the latest changes from the remote
git fetch origin

# Rebase your branch onto the latest dev
git rebase origin/dev
```

If there are conflicts, Git will pause and let you resolve them file by file:

```bash
# After resolving each conflict:
git add <resolved-file>
git rebase --continue

# To abort and go back to where you were:
git rebase --abort
```

> [!TIP]
> **Why rebase?** It rewrites your commits on top of the latest `dev`, producing a clean, linear history instead of merge commits. Your commits appear sequentially "after" the latest changes from `dev`.

---

### 4. Push Your Branch

After rebasing, push your branch to the remote. If you have already pushed before the rebase, you will need to force-push:

```bash
# First push
git push origin feature/your-descriptive-name

# After a rebase (if you previously pushed)
git push --force-with-lease origin feature/your-descriptive-name
```

> [!WARNING]
> Always use `--force-with-lease` (not `--force`) to avoid accidentally overwriting someone else's changes.

---

### 5. Create a Pull Request

1. Go to the repository on GitHub and click **"New pull request"**.
2. On the **"Compare changes"** page, set the branches:
   - **base:** `dev` *(the destination)*
   - **compare:** `feature/your-descriptive-name` *(your branch)*
3. Give the PR a clear title and description explaining what was changed and why.
4. Link any related issues or tasks.

---

### 6. Code Review

No code lands without review.

- 🤖 **Automated Review (Gemini):** When you open or update a PR, Gemini will automatically analyse your changes. Address every comment — fix the code or reply with a justification.
- 👤 **Human Review:** Once Gemini is satisfied, request a review from a teammate (e.g., Tobyn or Kalindi). They will check for correctness, readability, and architecture alignment.
- ✅ **Merge:** Once approved, the reviewer (or PR author) merges the PR into `dev`.

---

## 🏗️ Switching Tasks (Staggered Development)

If you finish a task, open a PR, and want to keep working without waiting for review, you have two options:

### Option A — Independent Tasks (Parallel Branches)

If your new task is **completely unrelated** to the un-merged PR, branch off `dev`:

```bash
git add -A && git commit -m "feat: finish task 1"
git checkout dev
git pull origin dev
git checkout -b feature/independent-task
```

### Option B — Dependent Tasks (Stacked Branches)

If your new task **depends on** the un-merged PR, branch off your current un-merged branch instead of `dev`:

```bash
# 1. Ensure you are on the completed, un-merged branch
git checkout feature/task-1-fix

# 2. Create a new branch directly from it
git checkout -b feature/task-2-new-feature

# 3. Work, commit, and push
git add -A && git commit -m "feat: add new feature based on task 1 fix"
git push origin feature/task-2-new-feature
```

**Handling the PR for stacked branches:**

- When creating the PR for `task-2`, set the **base branch** to `feature/task-1-fix`.
- Once `task-1` merges into `dev`, edit your `task-2` PR and change the base back to `dev`.
- Rebase `task-2` locally onto the updated `dev` and force-push:

```bash
git checkout feature/task-2-new-feature
git fetch origin
git rebase origin/dev
git push --force-with-lease origin feature/task-2-new-feature
```

---

## 🧹 Post-Merge Cleanup

Once merged, delete your feature branch remotely (via the GitHub UI) and locally:

```bash
git checkout dev
git pull origin dev
git branch -d feature/your-descriptive-name
```

> [!NOTE]
> If Git complains due to a squash merge, use `-D` to force delete.

---

## 🛠️ Repository Protections *(Admins Only)*

All Wildlife.ai repos should have the following settings configured in GitHub:

- **Default Branch:** Set to `dev`.
- **Protection Rules** for both `main` and `dev`:
  - Require a pull request before merging.
  - Require at least **1 approval**.
  - Restrict who can push directly — nobody should bypass PRs for these branches.

---

## 📌 Quick Reference Cheat-sheet

```
         main ← (release PRs only)
          ↑
         dev  ← (all feature PRs target here)
        / | \
 feature/ feature/ feature/
 task-a   task-b   task-c
```

```bash
# Start
git checkout dev && git pull origin dev
git checkout -b feature/my-task

# Commit
git add -A && git commit -m "feat: describe what you did"

# Rebase & Push
git fetch origin && git rebase origin/dev
git push origin feature/my-task                   # first time
git push --force-with-lease origin feature/my-task # after rebase

# Cleanup after merge
git checkout dev && git pull origin dev
git branch -d feature/my-task
```

---

Questions? Reach out to Victor or post in the team's Slack.

Ngā mihi nui for your contribution! 🙏
