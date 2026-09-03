# PR-Agent — AI code review for the wildlifeai org

Every enabled repo gets AI review on pull requests: an automatic review and
description when a PR opens, and on-demand commands anyone can type in a PR
comment. Runs on [PR-Agent](https://github.com/The-PR-Agent/pr-agent)
(community-owned, Apache 2.0) with **our own Google AI Studio (Gemini) key** —
no subscription, only API usage, and AI Studio's free tier covers our PR volume.

> Context: Google shut down the free Gemini Code Assist GitHub app on
> 17 July 2026. This is its replacement, under our control.

## How it fits together

- **`wildlifeai/.github/.github/workflows/pr-agent.yml`** — the reusable
  workflow: model choice, house rules (NZ English, wildlife/location-data care,
  no style nitpicks), what runs automatically.
- **A thin caller workflow in each repo** — triggers it on PR events and
  comments (template below; ~20 lines, no logic).
- **Org secret `GEMINI_API_KEY`** — one key for the whole org.

Changing the model or the house rules for every repo = one edit here.

## One-time org setup (admin)

1. Get an API key at [Google AI Studio](https://aistudio.google.com/apikey).
2. Add it as an **organisation** Actions secret named `GEMINI_API_KEY`:
   GitHub → wildlifeai org **Settings → Secrets and variables → Actions →
   New organization secret**, visibility "All repositories" (or selected).
   Or with the CLI (needs `admin:org` scope):
   `gh auth refresh -s admin:org` then
   `gh secret set GEMINI_API_KEY --org wildlifeai --visibility all`

## Enable it on a repo (two ways)

**A — starter workflow (easiest):** repo → **Actions → New workflow** → find
**"PR-Agent code review (Gemini)"** under the organisation templates → Commit.

**B — add the file directly:** create
`.github/workflows/pr-agent-review.yml` containing the template in
[`workflow-templates/pr-agent-review.yml`](../workflow-templates/pr-agent-review.yml).

## Using it

| You do | It does |
|---|---|
| Open a PR (or mark ready for review) | Automatic review + PR description |
| Comment `/review` | Fresh full review |
| Comment `/describe` | (Re)writes the PR title/description |
| Comment `/improve` | Concrete code-improvement suggestions |
| Comment `/ask <question>` | Answers about the PR, e.g. `/ask why change the retry logic?` |

Anyone who can comment on the PR can trigger it.

## Per-repo overrides

Add a `.pr_agent.toml` in a repo root to override any default, e.g.:

```toml
[pr_reviewer]
extra_instructions = "This is embedded C for the WW500 camera; flag any heap allocation."
```

## "Insufficient token budget to process" — files left out of a review

If the **PR Reviewer Guide** lists files it skipped, the diff didn't fit in the
budget PR-Agent gives the model. Two settings decide that budget, and the
*smaller* one wins:

| Setting | What it is | Ours |
|---|---|---|
| `config.max_model_tokens` | A hard cap PR-Agent applies to **every** model, whatever its real context window. Upstream default **32000** — this is the usual culprit. | `200000` |
| `config.custom_model_max_tokens` | Fallback window for models missing from PR-Agent's built-in table. Ignored for models it already knows (Gemini Flash is listed at 1M). | `1000000` |

Both are set in the reusable workflow, so raising the org-wide budget is one
edit there. Above the budget PR-Agent drops the extra context lines, then clips
oversized patches, then lists whatever is left as unprocessed.

A single repo can go higher (or lower) in its `.pr_agent.toml`:

```toml
[config]
max_model_tokens = 500000
```

Other ways to fit more of a PR in the budget:

- **Stop sending files nobody reviews.** Lockfiles, minified bundles, generated
  clients and vendored code eat the budget first. In a repo's `.pr_agent.toml`:

  ```toml
  [ignore]
  glob = ["**/package-lock.json", "**/*.min.js", "**/dist/**", "**/*.svg"]
  ```

- **Trim diff context** with `config.patch_extra_lines_before` (default 5) and
  `patch_extra_lines_after` (default 1) — only affects PRs that fit in one pass.
- **Split the PR.** Above ~200k tokens of diff, review quality drops well before
  the budget does; the model's attention is the real limit, not the window.

## Notes and limits

- **Forked PRs:** secrets don't flow to `pull_request` runs from forks, so the
  automatic review may skip external contributors' PRs. A maintainer commenting
  `/review` works (comment events run in the base repo with secrets).
- **Free-tier data caveat:** on AI Studio's free tier Google may use submitted
  content to improve products. Fine for our public repos; if a private repo
  handles sensitive material, use a paid-tier key (same secret, swap the key).
- **Quotas, not bills:** the free tier has no spend limit because it has no
  spend — Google's tiers table lists the Free tier's spend rate limit as N/A,
  and moving to a paid tier requires *us* to link a billing account. So raising
  the token budget cannot produce a charge on a free-tier key; over the limits
  the API returns 429 and the review fails. If reviews start failing with 429s,
  that's the quota — wait, split the PR, or upgrade.
- **The budget is a ceiling, not a serving size.** A 40-line PR still sends a
  40-line diff. Only big PRs consume more than they did under the old 32k cap,
  so day-to-day quota use barely moves.
- **Model:** `gemini-3.8-flash`, free of charge on the free tier and Google's
  strongest Flash model (their own description points it at software
  engineering work). Fallbacks are 3.5 Flash then 3.1 Flash Lite — rate limits
  are per-model, so a 429 on one still leaves the others.
- **Model pinning:** the reusable workflow tracks `The-PR-Agent/pr-agent@main`;
  pin to a release tag once we're happy with behaviour.
