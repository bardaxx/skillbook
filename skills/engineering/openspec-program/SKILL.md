---
name: openspec-program
description: Decomposes a PRD or large epic into a prioritized OpenSpec execution program and keeps slice lifecycle in sync with propose, apply, and archive. Use when a PRD exists but work is still one mega proposal, when breaking a feature into multiple OpenSpec changes, when picking the next slice, or when updating program status after OpenSpec steps.
---

# OpenSpec Program

Bridge **PRD / large epic** → **multiple OpenSpec changes** using a **program register** file. Do not replace OpenSpec CLI skills (`openspec-propose`, `openspec-apply-change`, `openspec-archive-change`); orchestrate them and keep the register current.

Register format and templates: [REFERENCE.md](REFERENCE.md). Register files must be timeline files named `openspec/TIMELINE_<context>.md`.

## When to use

| Situation | Action |
|-----------|--------|
| PRD exists, no execution plan | **Bootstrap** → `openspec/TIMELINE_<context>.md` |
| Need next unit of work | Pick highest-priority slice with status `Ready` |
| Slice is `Ready` | Delegate to `openspec-propose` (1:1 slice → change) |
| Change approved / implementing | Delegate to `openspec-apply-change`; set slice `Applying` |
| Change done | Delegate to `openspec-archive-change`; set slice `Archived` |
| Upstream only conversation | Create a `PRD.md` file first, then bootstrap |

## Orchestration

```
`PRD.md` file / tracked issue
    ↓
openspec-program → openspec/TIMELINE_<context>.md (concise slices)
    ↓ (per slice, status Ready)
openspec-propose → openspec/changes/<change-id>/
openspec-apply-change → implementation
openspec-archive-change → archive
    ↓
openspec-program → update slice status + progress log
```

## File layout

| Artifact | Path |
|----------|------|
| Timeline register | `openspec/TIMELINE_<context>.md` |
| Timeline config | `openspec/config.yaml` |
| Temporary program assets | `openspec/.temp_assets/` |

Program files are **planning registers**, not OpenSpec changes. Do **not** duplicate proposal/design/tasks content from `openspec/changes/` into the register.

Temporary working files used to prepare or apply slices (for example `audit.md`, checklists, scratch analysis notes, or intermediate outputs) must be stored under `openspec/.temp_assets/`.
These files are ephemeral support artifacts and must not be committed.

## Token budget and loading scope

Use `openspec/config.yaml` to keep context bounded. Default to these constraints unless the repo has stricter values:

- Mode: default timeline format (preferred).
- Load only:
  - active timeline register
  - selected slice
  - linked OpenSpec change
  - directly referenced files
- Do not load the full PRD unless the selected slice is ambiguous.
- Respect token ceilings from config before adding optional context.
- `openspec/config.yaml` is reference-only guidance for limits and loading; it must never be used to regenerate, rewrite, or recreate OpenSpec specs.

## Slice ↔ change

- **1:1** for implementable slices: each slice has a `Candidate OpenSpec change id` that becomes one change under `openspec/changes/`.
- **Exception:** doc-only / audit slices may use a single `audit-*` change.

### Change id naming rule (mandatory)

- Always prefix `Candidate OpenSpec change id` with the slice id lowercased, followed by the slice title slug.
- Format: `<slice-id-lower>-<slice-title-kebab>`.
- Example: `T10 - audit legacy test harness` -> `t10-audit-legacy-test-harness`.
- Keep `Spec link` aligned with the same id: `openspec/changes/<change-id>/`.
- If the slice title changes while the slice is still non-executed (`Ready` or `Spec Proposed`), update both the candidate id and spec link to keep them aligned.

## Slice IDs

| Prefix | Use |
|--------|-----|
| `F` | Product feature slices |
| `R` | Refactoring slices (no intended behavior change) |
| `T` | Testing and quality slices |
| `D` | Documentation-only slices |
| `I` | Infrastructure/tooling slices (optional) |

Number sequentially within prefix (`F01`, `R02`, …).

Prefix decision rules:

- Use `F` when user-visible behavior or API behavior changes.
- Use `R` for structural code improvements without intended behavior changes.
- Use `T` when the primary output is tests, coverage, or reliability harness work.
- Use `D` when the primary output is documentation/runbook/spec support text.
- Use `I` for CI, build, tooling, or environment changes.
- Split mixed slices into smaller slices when possible (for example `R` + `T`).

## Status model

Fixed lifecycle: `Ready` → `Spec Proposed` → `Applying` → `Applied` → `Archived`, plus `Blocked`.

| Transition | Register update | Delegate to |
|------------|-----------------|-------------|
| Pick slice | — | — |
| Change created | `Spec Proposed`; add change path | `openspec-propose` |
| Implementation started | `Applying`; branch/notes optional | `openspec-apply-change` |
| Code/tests verified | `Applied`; tests + validation commands | (during apply) |
| Change archived | `Archived`; archive path + date | `openspec-archive-change` |
| Cannot proceed | `Blocked`; open question + recommendation | — |

Follow the **Agent update checklist** in [REFERENCE.md](REFERENCE.md) at each step.

## Parallel execution policy

OpenSpec supports multiple open changes, but lifecycle transitions must stay deterministic.

- **Allow parallel proposals:** multiple slices can be in `Spec Proposed`.
- **Limit active implementation:** keep at most **2** slices in `Applying` at once.
- **Critical-area safeguard:** allow at most **1** `Applying` slice at a time for critical domains (for example payments, auth, checkout).
- **Keep `next` atomic:** one `next` command moves exactly one lifecycle gate for one slice.

Why this policy exists:

- reduces context switching and hidden queueing in review/CI
- limits merge conflicts and rework caused by long-lived concurrent branches
- improves throughput by finishing slices to `Archived` instead of accumulating half-done work
- keeps audit and rollback reasoning simple per slice/change

## Spec verification gate (mandatory)

Between `propose`, `apply`, and `archive`, always run the repository OpenSpec spec verification command and fix any issues before continuing.

- after `openspec-propose`: verify spec health before moving to `openspec-apply-change`
- after `openspec-apply-change`: verify spec health before moving to `openspec-archive-change`
- after `openspec-archive-change`: verify spec health before selecting the next slice

If verification fails, stop progression, resolve issues, re-run verification, then continue.

## Modes

### 1. Bootstrap

Create `openspec/TIMELINE_<context>.md` from a PRD or epic:

1. Read PRD/issue, the consumer repo’s `CONTEXT.md` (product terms), this skill’s [CONTEXT.md](CONTEXT.md) (workflow terms), and `AGENTS.md` (OpenSpec + GitNexus gates).
2. Choose `<context>` (short kebab-case or snake-case context label).
3. Create or update `openspec/config.yaml` with default mode and token/context limits.
4. Ensure `.gitignore` contains `openspec/.temp_assets/` (create/update it if needed).
5. Decompose into short, actionable slices.
6. Write timeline sections per [REFERENCE.md](REFERENCE.md): header, how-to, status model, compacted history, slices, dependency map, recommended execution order, agent checklist.
7. Link PRD at top (`PRD:` issue URL or path).
8. **Register the workflow in agent docs** — see [Agent documentation](#agent-documentation) below.
9. Do **not** run `openspec-propose` until user asks to start a slice.

**Parameters** (adapt per program):

- `scope` — what the program covers
- Priority scale — default P0 (urgent) / P1 / P2
- `principles` — optional global constraints (testing, security, etc.)
- `execution_order` — ordered slice list; note when reordered
- Item kind — `F` | `R` | `T` | `D` (`I` optional) per slice

### 2. Add slice / lifecycle

- **Add slice:** append one `### <ID> - Title` block using the default skeleton (see REFERENCE).
- **Update lifecycle:** change status and progress log only; link paths, do not copy OpenSpec artifacts.
- **Pick next:** highest priority among `Ready`, respecting execution order unless user overrides.
- **Reorder:** update Recommended Execution Order + short note why.

## Quick commands

Support lightweight command-style prompts that map to deterministic actions on an existing program register.

| Command | Intent | Required input | Expected action |
|---------|--------|----------------|-----------------|
| `status` | Show timeline/program state | none by default | Resolve active program and summarize slice counts by status, current blockers, and recommended next 1-2 slices. |
| `next:dry` | Preview the next OpenSpec gate | none by default | Resolve the active slice and return exactly one next gate action (`propose` or `apply` or `archive`) without executing it. |
| `next` | Execute one OpenSpec gate | none by default | Resolve the active slice and execute exactly one lifecycle gate: `Ready` -> run `openspec-propose`; `Spec Proposed` -> run `openspec-apply-change`; `Applied` -> run `openspec-archive-change`. Run OpenSpec spec verification after the gate, fix issues if any, update register, and stop. |
| `add "<feature description>"` | Add in-flight feature slice | feature intent only | Generate a compliant slice id and concise title, create a new slice block with minimum fields, and place it at the best point in execution order (not necessarily next), with a short rationale. |
| `add-next "<feature description>"` | Force immediate insertion | feature intent only | Generate a compliant slice id/title, create a new slice block, and place it as the next executable slice when valid; if dependencies block immediate insertion, place it at the earliest valid slot and mark it as forced in pipeline. |
| `start <slice-id>` | Start a specific slice | slice id | Validate slice exists and is actionable, then move lifecycle forward (or report blocker) and point to delegate skill. |
| `update <slice-id> "<feature delta>"` | Update scope of a planned slice | slice id + scope delta | Allowed only for non-executed slices (typically `Ready` or `Spec Proposed`): update goal/files/notes from the new intent, re-evaluate dependencies, and reorder execution as needed. |
| `block <slice-id>` | Mark work as blocked | slice id + blocking reason | Set `Blocked`, capture open question, and suggest an unblock path. |
| `deprecate <slice-id>` | Remove from active timeline safely | slice id + reason | Mark slice as deprecated in-place (without deleting history), remove it from active execution order, log replacement/follow-up if any, and reorder remaining queue when needed. |
| `restore <slice-id>` | Re-activate deprecated slice | slice id | Clear deprecation marker, choose lifecycle status (usually `Ready`), reinsert in execution order, and reorder if needed. |
| `reorder` | Re-prioritize queue | ordering rationale | Update Recommended Execution Order and add a short reason note. |

Command handling rules:

1. Resolve the active program from context by default.
2. If multiple timeline files are plausible candidates, ask the user to choose exactly once before proceeding.
3. Keep command responses concise and operational: current state, decision, and exact next action.
4. Never skip OpenSpec delegation steps: `openspec-propose` / `openspec-apply-change` / `openspec-archive-change`.
5. Always update the register after any command that changes lifecycle state.
6. For `add` and `add-next`, do not require user-provided slice id or title. Generate both from intent using the repository prefix/sequence rules and preserve uniqueness.
7. For `add`, do not append blindly to the end. Evaluate dependencies, risk, and leverage against existing slices, then insert the slice at the most suitable position in `Recommended Execution Order`.
8. When `add` or `add-next` causes reordering, explicitly report:
   - inserted position (for example "placed after R03, before F04")
   - concise rationale
   - whether it changes the next recommended slice
9. For `add-next`, insert as the nearest valid next position. If hard dependencies prevent immediate placement, do not force an invalid order; place it at the earliest valid slot, explain why, and add a short note that it was forced in pipeline.
10. For `update`, reject lifecycle-only edits; use it only for feature-scope updates on non-executed slices. If a slice is already `Applying`, `Applied`, or `Archived`, do not mutate scope and return a follow-up recommendation (for example create a new slice via `add`).
11. For `reorder`, `add`, `add-next`, `update`, `deprecate`, and `restore`, enforce folder-name consistency for non-applied slices only:
   - if execution order or slice title changes and a related `openspec/changes/<change-id>/` folder already exists for a slice in `Ready` or `Spec Proposed`, rename that folder to the new `<slice-id-lower>-<slice-title-kebab>` value
   - update `Candidate OpenSpec change id`, `Spec link`, and any in-register references accordingly
   - do not rename folders for `Applying`, `Applied`, or `Archived` slices
12. For `deprecate` and `restore`, run reorder logic whenever queue consistency or priority is impacted.
13. For `next`, execute one gate only and stop. Never chain propose + apply + archive in a single `next` call.
14. For `next`, stop immediately on blocker, failed verification/tests, or missing required approval; record progress up to the reached step and return the stop reason with the exact next manual action.
15. Do not chain propose + apply + archive in one command; keep explicit review checkpoints between gates.
16. After every `propose`, `apply`, and `archive`, run the repository OpenSpec spec verification command; if it reports issues, resolve and re-run before proceeding.
17. If temporary helper files are needed during program work, place them only under `openspec/.temp_assets/` and keep them out of version control via `.gitignore`.

Deprecation policy:

- Prefer **deprecate over delete** for slices that are no longer planned, to preserve auditability.
- Keep deprecated slices in the file with a clear marker in `Notes` and `Progress log` (include date and reason).
- Exclude deprecated slices from `Recommended Execution Order`; add a short note if they were replaced.
- Hard delete a slice only when the user explicitly asks for permanent removal.

## Per-slice minimum fields

Lite mode fields only: Status, Goal, Candidate OpenSpec change id, Spec link, Files to inspect, Notes, Progress log.

Do not switch to a full template by default. Keep timeline entries short and operational.

## Anti-overengineering gate

Do not use `openspec-program` for:

- bugfixes under 30 minutes
- isolated copy/UI tweaks
- one-file refactors
- exploratory spikes

## Repository agent instructions

Coding agents discover repo rules from **repository agent instruction files** (commonly `AGENTS.md`, sometimes additional entrypoints per repo). When this skill is **first adopted** in a repo, or when bootstrapping a program, ensure those files document the program layer — do not rely on the skill folder alone.

### When to update

| Trigger | Action |
|---------|--------|
| Skill copied into repo for the first time | Add or merge **OpenSpec Program** section in agent instruction files |
| Bootstrap of a new `openspec/TIMELINE_<context>.md` | Add active timeline pointer if the repo lists active programs |
| User asks only for lifecycle on an existing program | Update register only; skip instruction files unless section is missing |

### Files to patch

1. **Primary instruction file** (required) — usually `AGENTS.md`: extend `## OpenSpec Feature Workflow` or add the section from [REFERENCE.md](REFERENCE.md#agent-docs-snippet).
2. **Secondary entrypoints** (required when the repo uses them) — mirror the same OpenSpec Program block in every file that loads agent rules for that repo (e.g. tool-specific instruction files, `docs/agents/*.md`). Do not remove unrelated existing sections when inserting.
3. **One canonical wording** — pick the primary file as source of truth; keep mirrors aligned.

Use the **skill path actually installed** in routing tables (e.g. `.agents/skills/openspec-program/SKILL.md`, `skills/openspec-program/SKILL.md`).

### What instruction files must say

- Large PRD/epic work uses **`openspec-program`** before multiple `openspec-propose` calls.
- Registers live at `openspec/TIMELINE_<context>.md`.
- One implementable slice → one OpenSpec change; program file tracks status, not artifact content.
- After propose / apply / archive, update the register and delegate to the OpenSpec CLI skills.
- Timeline uses the default concise format.
- Token budget is enforced via `openspec/config.yaml`.

### Idempotency

- If the **OpenSpec Program** subsection already exists and matches, do not duplicate.
- If wording differs only slightly, reconcile to one canonical block and mirror to secondary entrypoints.
- List active timelines in instruction files only when useful (e.g. link `openspec/TIMELINE_<context>.md` in a short bullet).

Snippets and a routing table template: [REFERENCE.md — Agent docs snippet](REFERENCE.md#agent-docs-snippet).

## Repo gates (delegate, do not duplicate)

- **OpenSpec:** tests and production behavior changes require a change under `openspec/changes/` before implementation.
- **GitNexus:** explore + impact analysis before editing application symbols (see `AGENTS.md`).
- **Domain language:** product terms from the consumer repo’s `CONTEXT.md`; workflow terms from this skill’s [CONTEXT.md](CONTEXT.md).

## Related skills

| Skill | When |
|-------|------|
| `PRD.md` file | No formal PRD yet |
| `openspec-propose` | Slice `Ready` → create change |
| `openspec-apply-change` | Implement approved change |
| `openspec-archive-change` | After completion |
| `grill-with-docs` | Slice text needs domain alignment |
| `gitnexus-exploring` / `gitnexus-impact-analysis` | Before app symbol edits |

## Anti-patterns

- Using verbose, spec-like prose inside timeline files.
- One mega OpenSpec change for an entire PRD when slices are independent.
- Duplicating `proposal.md` / `design.md` / `tasks.md` content in the program file.
- Skipping register updates after propose / apply / archive.
- Installing the skill or bootstrapping a program without updating repository agent instructions — other sessions will miss the program layer.

## Additional resources

- Overview and intent (agents): [README.md](README.md)
- Workflow domain terms: [CONTEXT.md](CONTEXT.md)
- Program register template and checklists: [REFERENCE.md](REFERENCE.md)
