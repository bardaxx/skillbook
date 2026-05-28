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

Program files are **planning registers**, not OpenSpec changes. Do **not** duplicate proposal/design/tasks content from `openspec/changes/` into the register.

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

## Slice IDs

| Prefix | Use |
|--------|-----|
| `T` | Behavior / coverage slices |
| `L` | Legacy / stabilization / infrastructure |
| `F` | Product feature slices (when needed) |

Number sequentially within prefix (`T01`, `L02`, …).

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

## Modes

### 1. Bootstrap

Create `openspec/TIMELINE_<context>.md` from a PRD or epic:

1. Read PRD/issue, `CONTEXT.md` (domain terms), and `AGENTS.md` (OpenSpec + GitNexus gates).
2. Choose `<context>` (short kebab-case or snake-case context label).
3. Create or update `openspec/config.yaml` with default mode and token/context limits.
4. Decompose into short, actionable slices.
5. Write timeline sections per [REFERENCE.md](REFERENCE.md): header, how-to, status model, compacted history, slices, dependency map, recommended execution order, agent checklist.
6. Link PRD at top (`PRD:` issue URL or path).
7. **Register the workflow in agent docs** — see [Agent documentation](#agent-documentation) below.
8. Do **not** run `openspec-propose` until user asks to start a slice.

**Parameters** (adapt per program):

- `scope` — what the program covers
- Priority scale — default P0 (urgent) / P1 / P2
- `principles` — optional global constraints (testing, security, etc.)
- `execution_order` — ordered slice list; note when reordered
- Item kind — `T` | `L` | `F` per slice

### 2. Add slice / lifecycle

- **Add slice:** append one `### <ID> - Title` block using the default skeleton (see REFERENCE).
- **Update lifecycle:** change status and progress log only; link paths, do not copy OpenSpec artifacts.
- **Pick next:** highest priority among `Ready`, respecting execution order unless user overrides.
- **Reorder:** update Recommended Execution Order + short note why.

## Quick commands

Support lightweight command-style prompts that map to deterministic actions on an existing program register.

| Command | Intent | Required input | Expected action |
|---------|--------|----------------|-----------------|
| `status` | Show timeline/program state | Program path or slug | Summarize slice counts by status, current blockers, and recommended next 1-2 slices. |
| `next` | Advance to next unit of work | Program path or slug | Pick highest-priority `Ready` slice (respect execution order unless user override), then propose the exact next command (usually `openspec-propose`). |
| `add <slice-id> "<title>"` | Add in-flight feature slice | Program + minimal slice details | Append a new slice block with minimum required fields, set initial status (usually `Ready`), and place it in execution order. |
| `start <slice-id>` | Start a specific slice | Program + slice id | Validate slice exists and is actionable, then move lifecycle forward (or report blocker) and point to delegate skill. |
| `update <slice-id> <status>` | Manual lifecycle update | Program + slice id + target status | Apply valid lifecycle transition, update progress log, and record linked artifact paths. |
| `block <slice-id>` | Mark work as blocked | Program + slice id + blocking reason | Set `Blocked`, capture open question, and suggest an unblock path. |
| `deprecate <slice-id>` | Remove from active timeline safely | Program + slice id + reason | Mark slice as deprecated in-place (without deleting history), remove it from active execution order, and log replacement/follow-up if any. |
| `restore <slice-id>` | Re-activate deprecated slice | Program + slice id | Clear deprecation marker, choose lifecycle status (usually `Ready`), and reinsert in execution order. |
| `reorder` | Re-prioritize queue | Program + ordering rationale | Update Recommended Execution Order and add a short reason note. |

Command handling rules:

1. If no program is provided, resolve active program from context; if ambiguous, ask for the target file once.
2. Keep command responses concise and operational: current state, decision, and exact next action.
3. Never skip OpenSpec delegation steps: `openspec-propose` / `openspec-apply-change` / `openspec-archive-change`.
4. Always update the register after any command that changes lifecycle state.

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
- **Domain language:** use terms from `CONTEXT.md` in slice text and when invoking propose.

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
- Program register template and checklists: [REFERENCE.md](REFERENCE.md)
