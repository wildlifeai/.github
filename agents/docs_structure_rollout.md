# Adopting the agent + documentation structure in a repository

*Recipe for bringing any wildlifeai repository to the shared structure described in
[`README.md`](README.md): agent guides that load themselves, a home for developer
discussions, and findings that end up on a project board instead of in someone's inbox.
Budget 30–60 minutes depending on what the repo already has.*

## The five elements

| # | Element | Purpose |
|---|---|---|
| 1 | **`AGENTS.md`** (root) + **`CLAUDE.md`** containing exactly `@AGENTS.md` | The entry point any agent (or new developer) reads first: what the repo is, how to build/run/test, the non-negotiables, and a map of the documentation. `AGENTS.md` is the cross-tool convention; `CLAUDE.md` is what Claude Code auto-loads. |
| 2 | **`.agents/skills/SKILL.md`** | The deep layer: workflow rules, guardrails, repo-specific traps, cross-repo contracts. `AGENTS.md` directs agents here before any change. Further skills get their own folder alongside it. |
| 3 | **`documentation/development reports/`** with a process **README** | Dated threads holding the working material of design discussions and reviews. |
| 4 | **`.github/ISSUE_TEMPLATE/review-finding.md`** + project-board auto-add | Filing a finding takes 30 seconds and lands on the board with no manual step. |
| 5 | **`.gitignore` hygiene** | Shared agent files (`.agents/`, `AGENTS.md`, `CLAUDE.md`) must be committable; personal tool configs (`.claude/`, `.cursorrules`, `.windsurfrules`) stay ignored. |

## Two rules that make it work

1. **Docs are the record; issues are the tracker.** Anything still open when a discussion
   pauses — a bug, an undecided question, a follow-up — becomes a GitHub issue. A
   document is never the only place an open item lives.
2. **Every thread README starts with Status / Outcome / Open items.** The Outcome is the
   short summary a future developer reads *instead of* the thread; Open items are issue
   links, nothing else.

A thread closes when three boxes are ticked: outcome written, open items filed as issues,
affected topic docs updated. That last box is what keeps durable how-to knowledge out of
conversation threads and in the documentation where people look for it.

## Steps

1. **Write `.agents/skills/SKILL.md`.** Distil it from what the team already knows:
   existing onboarding docs, the README, and the things new contributors always get
   wrong. Good candidates: build/test invariants, deployment or release rules, ownership
   boundaries between repos, data or API contracts that cannot be changed unilaterally,
   and the traps that have bitten someone.
2. **Write `AGENTS.md`** (keep it to a page: identity, commands, non-negotiables, doc map)
   and **`CLAUDE.md`** with the single line `@AGENTS.md`.
3. **Create `documentation/development reports/README.md`** carrying the two rules and
   the closing checklist. If working documents already exist, list them as threads with a
   status each rather than reorganising them.
4. **Add the issue template**, then enable auto-add on your project board
   (project → ⋯ → Workflows → *Auto-add to project*, filter `repo:<org>/<repo> is:issue`).
5. **Check `.gitignore`** — run `git check-ignore .agents/skills/SKILL.md CLAUDE.md AGENTS.md`;
   it should print nothing. Older repos often ignore `.agents/` or `CLAUDE.md` from the
   days when agent files were personal.

## Definition of done

- A fresh agent session self-orients: `CLAUDE.md`/`AGENTS.md` load, point at the skill,
  the skill points at the documentation.
- `documentation/development reports/README.md` exists and lists any existing threads.
- Filing a finding takes 30 seconds and appears on the board without manual steps.
- Nothing shared is gitignored.

## Reference implementation

[Seeed_Grove_Vision_AI_Module_V2](https://github.com/wildlifeai/Seeed_Grove_Vision_AI_Module_V2)
— agent files in [PR #150](https://github.com/wildlifeai/Seeed_Grove_Vision_AI_Module_V2/pull/150),
development-reports convention and issue template on branch `review/cgp-141` (thread
`_Documentation/development reports/2026-07_pr141-review-cgp/`). The READMEs and template
transfer nearly verbatim; `AGENTS.md` and `SKILL.md` are repo-specific by nature.
