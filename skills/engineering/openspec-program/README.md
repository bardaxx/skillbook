# OpenSpec Program

Agent skill that sits **between a large PRD and many small OpenSpec changes**.

When you receive a PRD or epic that would normally become one oversized OpenSpec proposal, use this skill first: decompose the work, track slices in a program register, then delegate each slice to the existing OpenSpec skills (`openspec-propose`, `openspec-apply-change`, `openspec-archive-change`). You orchestrate and keep state in sync; you do not replace the OpenSpec CLI.

## The gap you are filling

OpenSpec gates a **single** change well: proposal, design, tasks, implement, archive.

A PRD or epic is usually **many** changes — different priority, risk, and scope. Without a program layer, you will either:

- propose one mega change that is hard to review and resume, or
- lose track of which slices were proposed, applied, or archived.

This skill adds a **program register**: a markdown file listing slices and status. Each slice still gets its own change under `openspec/changes/`.

## Your responsibilities

| Situation | What to do |
|-----------|------------|
| PRD exists, no execution plan | **Bootstrap** → `openspec/TIMELINE_<context>.md` |
| Operator asks what is next | Pick highest-priority slice with status `Ready` |
| Slice is `Ready` | Run `openspec-propose` (1:1 slice → change id) |
| Change approved / implementing | Run `openspec-apply-change`; set slice `Applying` |
| Change complete | Run `openspec-archive-change`; set slice `Archived` |
| No formal PRD yet | Create a `PRD.md` file (or equivalent), then bootstrap |

The register is a **map and status board**, not a copy of `proposal.md` or `tasks.md`. Link paths; do not duplicate OpenSpec artifacts.

## Workflow

```
PRD or tracked issue
        ↓
openspec-program  →  openspec/TIMELINE_<context>.md
        ↓
For each slice marked Ready:
        openspec-propose  →  openspec/changes/<id>/
        openspec-apply-change
        openspec-archive-change
        ↓
Update the program register after each step
```

You act as **program coordinator**: decomposition and bookkeeping only.

## How to use command-style prompts

The skill supports concise command-style prompts that map to deterministic register actions.

Common commands:

- `status`: summarize counts by status, blockers, and top recommended next slices for the active program.
- `next:dry`: resolve the active slice and show exactly one next gate action (`propose` or `apply` or `archive`) without executing it.
- `next`: execute exactly one OpenSpec gate according to lifecycle state, then stop and update the register.
- `add "<feature description>"`: add a new slice from intent. The skill generates slice id/title, fills minimum fields, and inserts it at the best position in execution order with rationale.
- `add-next "<feature description>"`: same as `add`, but force placement as the next executable work item when valid. If dependencies prevent immediate placement, place it at the earliest valid slot and record that it was forced.
- `start <slice-id>`: move a selected slice into the next actionable lifecycle step.
- `update <slice-id> "<feature delta>"`: update feature scope for a non-executed slice (for example `A+B` -> `A+B+C`), then re-evaluate dependencies and execution order.
- `block <slice-id> "<reason>"`: mark a slice as blocked with recommendation.
- `deprecate <slice-id> "<reason>"`: remove slice from active plan without deleting history, then reorder if needed.
- `restore <slice-id>`: reactivate a deprecated slice, reinsert in execution order, and reorder if needed.
- `reorder`: update Recommended Execution Order with a rationale note.

Notes:

- Prefer deprecating over deleting to preserve audit history.
- Any lifecycle-changing command must update the register.
- OpenSpec delegation remains mandatory (`openspec-propose` / `openspec-apply-change` / `openspec-archive-change`).
- Program selection is implicit by default. Ask for an explicit target only when multiple timeline files are plausible candidates.

## Example workflow with commands

Scenario: a PRD was decomposed into `openspec/TIMELINE_public-api-hardening.md`.

1. Check current state
   - Prompt: `status`
   - Outcome: identifies `T03` as top `Ready` slice and highlights one blocked legacy item.

2. Preview the next gate
   - Prompt: `next:dry`
   - Outcome: selects `T03` and shows the single next gate (for example: run `openspec-propose`).

3. Execute one gate
   - Prompt: `next`
   - Outcome: runs only one gate (for example `openspec-propose`), updates lifecycle/progress, then stops for human review.

4. Continue with explicit gates
   - Prompt: repeated `next`
   - Outcome: each call advances a single gate and preserves review checkpoints between propose, apply, and archive.

5. Add a new in-flight feature request
   - Prompt: `add "Add rate-limit visibility endpoints"`
   - Register update: generate a new slice id/title, add the slice with minimum fields, and place it in the best execution-order position (not always next), with a short rationale.

6. Force-add a feature as next work
   - Prompt: `add-next "Expose rate-limit headers in responses"`
   - Register update: generate a new slice id/title and place it as next executable slice when valid; otherwise place at earliest valid slot and mark it as forced in pipeline.

7. Update a non-executed slice scope
   - Prompt: `update F05 "Also include per-tenant limits and audit logging"`
   - Register update: update goal/files/notes for `F05`, then re-check dependencies and reorder execution when needed.

8. Deprecate outdated work safely
   - Prompt: `deprecate R02 "Replaced by F05 scope"`
   - Register update: keep `R02` in file, add deprecation note/date, remove from active execution order, and reorder remaining queue if needed.

9. Continue flow
   - Prompt: `status`
   - Outcome: updated queue, active blockers, and next recommended slice.

## Command guide (updated)

Use this as a quick operational reference.

| Command | Use when | Input style | What it does |
|---------|----------|-------------|--------------|
| `status` | You need current program visibility | no args | Resolves active timeline and reports counts by status, blockers, and next recommended slices. |
| `next:dry` | You want a preview before execution | no args | Resolves the active slice and returns one next gate action (`propose` or `apply` or `archive`) without executing it. |
| `next` | You want controlled progress with review points | no args | Executes exactly one gate based on current state, updates register, and stops. Use repeated `next` calls to walk propose -> apply -> archive with human checkpoints. |
| `add "<feature description>"` | New feature work appears mid-program | free-form feature intent | Generates slice id and title, creates the slice with minimum fields, evaluates dependencies, and inserts it in the best execution position. |
| `add-next "<feature description>"` | New feature is urgent and should run next | free-form feature intent | Same as `add`, but tries to place the new slice as next executable item. If blocked by dependencies, places at earliest valid slot and marks it as forced in pipeline. |
| `start <slice-id>` | You need to move a specific slice into active execution | existing slice id | Advances the selected slice to the next actionable lifecycle state and points to required OpenSpec delegation steps. |
| `update <slice-id> "<feature delta>"` | Planned scope changed before execution | slice id + scope delta | Allowed only on non-executed slices. Updates scope fields, re-checks dependencies, and reorders queue if needed. |
| `block <slice-id> "<reason>"` | Progress cannot continue | slice id + blocker reason | Sets `Blocked`, captures reason, and records a suggested unblock path. |
| `deprecate <slice-id> "<reason>"` | A slice is no longer needed | slice id + reason | Keeps slice history, marks deprecation, removes it from active queue, and runs reorder logic when needed. |
| `restore <slice-id>` | A deprecated slice becomes relevant again | deprecated slice id | Re-activates the slice, reinserts it in execution order, and reorders queue when needed. |
| `reorder` | Priority/dependencies changed globally | optional rationale | Recomputes `Recommended Execution Order` and records concise rationale for queue changes. |

Behavior rules:

- Program selection is implicit; ask only when multiple timeline candidates exist.
- `next` executes one gate per call; `next:dry` previews one gate.
- `update` is for scope evolution, not lifecycle-only status edits.
- `deprecate` and `restore` include reorder checks automatically.

## Parallelism and WIP

Parallel implementation is supported, but intentionally constrained:

- multiple slices can be in `Spec Proposed` at the same time
- keep at most **2** slices in `Applying` globally
- keep at most **1** `Applying` slice in critical domains (for example checkout, auth, payments)
- keep `next` atomic (one gate for one slice) to maintain traceability

This improves flow by reducing context switching, review/CI queue congestion, and merge conflict pressure while increasing completed slices (`Archived`) per cycle.

## Paths you should know

| Path | Role |
|------|------|
| `openspec/TIMELINE_<context>.md` | Program timeline register (required naming) |
| `openspec/config.yaml` | Token limits, loading scope, and timeline defaults |
| `openspec/changes/<id>/` | OpenSpec artifacts per slice |

Slice prefixes: **F** (feature), **R** (refactoring), **T** (testing), **D** (documentation), optional **I** (infrastructure/tooling). Number sequentially (`F01`, `R02`, …).

Slice lifecycle: `Ready` → `Spec Proposed` → `Applying` → `Applied` → `Archived`, plus `Blocked`.

## Default operating profile

- Use concise timeline entries by default.
- Keep each slice short and operational (goal, spec link, files, status, short notes).
- Enforce context loading from `openspec/config.yaml`:
  - active timeline
  - selected slice
  - linked OpenSpec change
  - directly referenced files
- Do not load full PRD unless the slice is ambiguous.
- Treat `openspec/config.yaml` as reference-only guidance; never use it to recreate or rewrite OpenSpec specs.
- Do not use this flow for small work (quick bugfixes, one-file refactors, isolated UI copy edits, exploratory spikes).

## Repository setup (first use)

When this skill is adopted in a repo, document the program layer in **repository agent instructions** so every agent session sees the same rules — typically `AGENTS.md`, or whatever entrypoint files that repo uses for agent guidance.

On bootstrap:

1. Create or update the program register.
2. Add or merge an **OpenSpec Program** section in those instruction files (snippets in [REFERENCE.md](REFERENCE.md#agent-docs-snippet)).
3. Point to the installed skill path in any task → skill routing table.

Do not assume other agents will read this folder unless the repo instructions tell them to.

## Files in this skill folder

| File | Use |
|------|-----|
| [SKILL.md](SKILL.md) | Procedures, modes, gates — your primary instruction set |
| [REFERENCE.md](REFERENCE.md) | Register template, agent-doc snippets, checklists |
| [TIMELINE_SKELETON.md](TIMELINE_SKELETON.md) | Copy-ready default timeline skeleton |
| [README.md](README.md) | Intent and mental model (this file) |

## Example prompts you should handle

**Decompose a PRD**  
Bootstrap `openspec/TIMELINE_<context>.md`, add concise default slices with goals and candidate change ids, set execution order, register repo agent instructions, then wait before proposing unless asked to start.

**What is next?**  
Read the program file, select the highest-priority `Ready` slice, invoke `openspec-propose` with that slice’s target behavior and change id.

**Change archived**  
Set slice to `Archived`, update progress log with archive path and date, surface the next `Ready` slice.

## Out of scope for this skill

- Replacing `openspec-propose`, apply, or archive.
- Storing full OpenSpec content in the program register.
- Writing verbose, spec-like prose in timeline files.

Full procedures: [SKILL.md](SKILL.md). Templates and instruction-file snippets: [REFERENCE.md](REFERENCE.md).
