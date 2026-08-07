# Agent guides across wildlife.ai repositories

How AI coding agents (and new developers) get oriented in our repos, and where the
rules live. Adopted Aug 2026; the firmware repo
([Seeed_Grove_Vision_AI_Module_V2](https://github.com/wildlifeai/Seeed_Grove_Vision_AI_Module_V2))
is the reference implementation.

## Per-repo files (committed, auto-loaded)

Every repository carries three files:

| File | Purpose |
|---|---|
| `AGENTS.md` (root) | Quickstart: what the repo is, build/test commands, non-negotiables, doc map. The cross-tool entry point. |
| `CLAUDE.md` (root) | One line — `@AGENTS.md` — so Claude Code auto-loads the same guide. |
| `.agents/skills/SKILL.md` | The deep layer: workflow rules, guardrails, repo-specific traps, cross-repo contracts. `AGENTS.md` directs agents here before any change. |

These are **committed**, never copied to home directories — a committed file cannot
drift from the team's actual rules, and every agent session picks it up with zero
setup. Personal tool configs (`.claude/`, `.cursorrules`, `.windsurfrules`) stay
gitignored.

Rolling this out to a repo that doesn't have it yet:
[`docs_structure_rollout.md`](docs_structure_rollout.md) — verified per-repo checklists
for website, backend, mobile and hardware, including the developer-discussion
(`documentation/development reports/`) and issue-tracking conventions that go with it.

## Org-wide skills (this folder)

Skills that apply to every repo live here and are **referenced in place** (a repo's
`AGENTS.md`/`SKILL.md` links to them), not duplicated:

| Skill | Covers |
|---|---|
| [`git-SKILL.md`](git-SKILL.md) | Git discipline: fetch-first, no plain force-push, branch/PR workflow, stacked PRs |

## Precedence

Org skills are the default. A repo's own `AGENTS.md`/`SKILL.md` may **tighten** them
for local realities, and the repo rule wins — e.g. the firmware repo forbids rebasing
or force-pushing (even `--force-with-lease`) its stacked feature branches while they
are under external review, which is stricter than `git-SKILL.md`'s general rule.
