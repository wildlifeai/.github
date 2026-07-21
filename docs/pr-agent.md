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

## Notes and limits

- **Forked PRs:** secrets don't flow to `pull_request` runs from forks, so the
  automatic review may skip external contributors' PRs. A maintainer commenting
  `/review` works (comment events run in the base repo with secrets).
- **Free-tier data caveat:** on AI Studio's free tier Google may use submitted
  content to improve products. Fine for our public repos; if a private repo
  handles sensitive material, use a paid-tier key (same secret, swap the key).
- **Quotas:** free-tier Gemini Flash allows a healthy number of requests/day;
  if reviews ever start failing with 429s, that's the quota — wait or upgrade.
- **Model pinning:** the reusable workflow tracks `The-PR-Agent/pr-agent@main`;
  pin to a release tag once we're happy with behaviour.
