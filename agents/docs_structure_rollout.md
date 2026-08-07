# Aligning documentation, developer discussions and agent guides across the wildlifeai repos

*Aug 2026. The firmware repo (Seeed_Grove_Vision_AI_Module_V2) now runs a documentation
system the other repos partially pioneered; this page is the recipe for bringing
**ww-website, ww-backend, ww-mobile-app and ww-hardware** to the same shape. Each repo
section below states what already exists (verified against the working copies, Aug 2026)
and exactly what to create. Budget ~30–60 minutes per repo.*

## The target — five elements per repo

| # | Element | Purpose |
|---|---|---|
| 1 | **`AGENTS.md`** (root) + **`CLAUDE.md`** containing exactly `@AGENTS.md` | The auto-loaded entry point for any coding agent (and a fine 2-minute human orientation): what the repo is, build/test commands, non-negotiables, doc map. Claude Code loads `CLAUDE.md`; other tools read `AGENTS.md`. |
| 2 | **`.agents/skills/SKILL.md`** | The deep layer: workflow rules, guardrails, repo-specific traps, cross-repo contracts. `AGENTS.md`'s first instruction is "read this before changing anything". |
| 3 | **`documentation/development reports/`** with a process **README** | Dated discussion threads (reviews, proposals, evidence). Rule 1: *docs are the record, GitHub issues are the tracker* — an open item lives on the project board, never only in a document. Rule 2: every thread README keeps **Status / Outcome / Open items** current, with a 3-box closing checklist. |
| 4 | **`.github/ISSUE_TEMPLATE/review-finding.md`** + the board's **auto-add workflow** | 30-second filing of findings; the org project board sweeps them up automatically. |
| 5 | **`.gitignore` hygiene** | The *shared* agent files (`.agents/`, `AGENTS.md`, `CLAUDE.md`) must be committable; *personal* configs (`.claude/`, `.cursorrules`, `.windsurfrules`, `**/claude/`) stay ignored. |

**Reference implementation** — copy and adapt from the firmware repo:
- `AGENTS.md`, `CLAUDE.md`, `.agents/skills/SKILL.md`: branch `docs/agents-and-skills` ([PR #150](https://github.com/wildlifeai/Seeed_Grove_Vision_AI_Module_V2/pull/150))
- `_Documentation/development reports/README.md` (process), thread example `2026-07_pr141-review-cgp/`, and `.github/ISSUE_TEMPLATE/review-finding.md`: branch `review/cgp-141`

The READMEs and template transfer almost verbatim (adjust paths and the board link).
`AGENTS.md`/`SKILL.md` are repo-specific by nature — distil them from each repo's
existing onboarding/resources docs; the skill is the home for the tribal knowledge that
otherwise lives in one person's head.

---

## ww-website — closest to done (~30 min)

Already there: `documentation/{README.md, development reports, onboarding, resources}`;
`.agents/skills/SKILL.md` (committed, and already excellent), `.agents/skills/guide-author/`,
`.agents/DESIGN.md`.

- [ ] Add root `AGENTS.md` (quickstart distilled from the existing SKILL.md: stack, dev
      commands, the Database Ownership Invariant, doc map) and one-line `CLAUDE.md`.
- [ ] Add `documentation/development reports/README.md` (the process file — the folder
      exists but carries no rules; move the existing reports' status into a thread table).
- [ ] Add `.github/ISSUE_TEMPLATE/review-finding.md`.
- [ ] Board: verify an auto-add workflow exists for this repo (the org board currently
      shows WW-grove_vision, WW-mobile-app, WW-API, WW_metadata, WW-sandpit — **website
      appears to be missing**; add via project ⋯ → Workflows → Auto-add, filter `is:issue`).

## ww-backend — mostly layout alignment (~30 min)

Already there: `documentation/{development reports, onboarding, resources}`;
`.agents/SKILL.md` **tracked** at the `.agents/` root.

- [ ] Move `.agents/SKILL.md` → `.agents/skills/SKILL.md` (matches website + firmware;
      leaves room for further skills).
- [ ] Fix `.gitignore` line ~237: it ignores `.agents` even though the skill is tracked —
      delete the line (and keep/add ignores for the *personal* configs instead).
- [ ] Add root `AGENTS.md` + `CLAUDE.md` (quickstart: services, schema-ownership rule —
      this repo owns `supabase/schemas/`, the counterpart of the website's invariant —
      migration workflow, test commands).
- [ ] Add `documentation/development reports/README.md` + the issue template.
- [ ] Board: confirm whether the existing **WW-API** auto-add workflow covers this repo;
      if it points elsewhere, add one for ww-backend.

## ww-mobile-app — needs the whole agent layer (~60 min)

Already there: `documentation/{development reports, onboarding, resources}` with strong
content (`DOCUMENTATION-AUDIT.md`, the transfer proposals/specs in development reports);
board auto-add **already enabled** (WW-mobile-app). No `.agents/`, no `AGENTS.md`/`CLAUDE.md`.

- [ ] Write `.agents/skills/SKILL.md` — good sources: `DOCUMENTATION-AUDIT.md`, the
      onboarding folder, and the BLE-protocol knowledge in
      `development reports/` (sliding-window transfer spec). Candidate guardrails: BLE
      command registry as the single source of truth (`commandRegistry.ts`), the `AI `
      command prefix contract, op-parameter index mirror with the firmware, release/build
      channels.
- [ ] Add root `AGENTS.md` + `CLAUDE.md` (build/run commands, test command, the firmware
      contract warning).
- [ ] Add `documentation/development reports/README.md` (fold the existing four reports
      into the thread table with a status each) + the issue template.
- [ ] Check `.gitignore` doesn't block `.agents/`/`CLAUDE.md` (and note: the file has a
      non-UTF-8 byte partway through — worth a quick clean while in there).

## ww-hardware — greenfield (~60 min)

Nothing yet: no `documentation/`, no agent files, and `.gitignore` lines 13/18 ignore
`.agents/` and `CLAUDE.md`. Existing technical docs live in
`MokoTech/Workspace/_Documentation/` (BLE CI, DFU guides) and stay where they are.

- [ ] Create top-level `documentation/development reports/README.md` (same process file;
      point it at `MokoTech/Workspace/_Documentation/` for topic docs). First thread
      candidate: the PR #27 fast-transfer review — `BLE_Fast_File_Transfer.md` and
      `REVIEW_PR27.md` already exist on `feature/ble-fast-transfer` and belong in it.
- [ ] Remove the two `.gitignore` lines; keep personal-config ignores.
- [ ] Write `.agents/skills/SKILL.md` — candidate content: MokoTech/nRF SDK build
      (SES project, `version.mk` release flow), DFU update paths (Nordic DFU, the
      hex/zip artifact naming `WildlifeWatcher_1_ww500_c02_nus_NNNNNN`), the
      **`aiProcessor.h` op-parameter mirror contract with the firmware repo** (never
      change unilaterally), stacked-branch rules, CI upload gating (feature pushes must
      not trigger production uploads).
- [ ] Add root `AGENTS.md` + `CLAUDE.md`, and the issue template.
- [ ] Board: add an auto-add workflow for ww-hardware (currently absent).

---

## Definition of done (any repo)

1. A fresh agent session in the repo self-orients: `CLAUDE.md`/`AGENTS.md` load, point
   at the skill, the skill points at the docs.
2. `documentation/development reports/README.md` exists; every existing working doc is
   either listed in its thread table or left in `resources/`.
3. Filing a finding takes 30 seconds (template) and appears on the
   [project board](https://github.com/orgs/wildlifeai/projects/3) without manual steps.
4. `git check-ignore .agents/skills/SKILL.md CLAUDE.md AGENTS.md` returns nothing.

Questions or improvements: thread them in your repo's `development reports/` and file
issues — that's the system demonstrating itself.
