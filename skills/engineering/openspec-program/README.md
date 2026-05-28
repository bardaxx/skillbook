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
| PRD exists, no execution plan | **Bootstrap** → `openspec/programs/<slug>.md` |
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
openspec-program  →  openspec/programs/<slug>.md
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

- `status <program>`: summarize counts by status, blockers, and top recommended next slices.
- `next <program>`: pick the highest-priority `Ready` slice (respecting execution order).
- `add <program> <slice-id> "<title>"`: append a new slice with minimum required fields.
- `start <program> <slice-id>`: move a selected slice into the next actionable lifecycle step.
- `update <program> <slice-id> <status>`: apply a lifecycle update and refresh progress log.
- `block <program> <slice-id> "<reason>"`: mark a slice as blocked with recommendation.
- `deprecate <program> <slice-id> "<reason>"`: remove slice from active plan without deleting history.
- `restore <program> <slice-id>`: reactivate a deprecated slice and reinsert it in execution order.
- `reorder <program>`: update Recommended Execution Order with a rationale note.

Notes:

- Prefer deprecating over deleting to preserve audit history.
- Any lifecycle-changing command must update the register.
- OpenSpec delegation remains mandatory (`openspec-propose` / `openspec-apply-change` / `openspec-archive-change`).

## Example workflow with commands

Scenario: a PRD was decomposed into `openspec/programs/public-api-hardening.md`.

1. Check current state
   - Prompt: `status public-api-hardening`
   - Outcome: identifies `T03` as top `Ready` slice and highlights one blocked legacy item.

2. Move to the next slice
   - Prompt: `next public-api-hardening`
   - Outcome: selects `T03`, suggests running `openspec-propose` with `t03-public-api-validation`.

3. Create the spec for that slice
   - Delegate: `openspec-propose` for `T03`
   - Register update: set `T03` to `Spec Proposed`, add `openspec/changes/t03-public-api-validation/`.

4. Start implementation
   - Prompt: `start public-api-hardening T03`
   - Delegate: `openspec-apply-change`
   - Register update: set `T03` to `Applying`, add branch/agent context.

5. Mark implementation completed
   - Prompt: `update public-api-hardening T03 Applied`
   - Register update: add validation commands and key changed files.

6. Archive completed change
   - Delegate: `openspec-archive-change`
   - Register update: set `T03` to `Archived`, add archive path and date.

7. Add a new in-flight feature request
   - Prompt: `add public-api-hardening F05 "Add rate-limit visibility endpoints"`
   - Register update: append `F05` slice and insert it in execution order with a short scope-change note.

8. Deprecate outdated work safely
   - Prompt: `deprecate public-api-hardening L02 "Replaced by F05 scope"`
   - Register update: keep `L02` in file, add deprecation note/date, remove from active execution order.

9. Continue flow
   - Prompt: `status public-api-hardening`
   - Outcome: updated queue, active blockers, and next recommended slice.

## Paths you should know

| Path | Role |
|------|------|
| `openspec/programs/<slug>.md` | Program register (preferred for new work) |
| `openspec/TIMELINE_TEST.md` | Legacy example of the same register shape |
| `openspec/changes/<id>/` | OpenSpec artifacts per slice |

Slice prefixes: **T** (behavior/tests), **L** (legacy/stabilization), **F** (product features). Number sequentially (`T01`, `L02`, …).

Slice lifecycle: `Ready` → `Spec Proposed` → `Applying` → `Applied` → `Archived`, plus `Blocked`.

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
| [README.md](README.md) | Intent and mental model (this file) |

## Example prompts you should handle

**Decompose a PRD**  
Bootstrap `openspec/programs/<slug>.md`, add slices with goals and candidate change ids, set execution order, register repo agent instructions, then wait before proposing unless asked to start.

**What is next?**  
Read the program file, select the highest-priority `Ready` slice, invoke `openspec-propose` with that slice’s target behavior and change id.

**Change archived**  
Set slice to `Archived`, update progress log with archive path and date, surface the next `Ready` slice.

## Out of scope for this skill

- Replacing `openspec-propose`, apply, or archive.
- Storing full OpenSpec content in the program register.
- Using “timeline” as the product name — the register is an implementation detail; the idea is **program decomposition from a PRD**.

Full procedures: [SKILL.md](SKILL.md). Templates and instruction-file snippets: [REFERENCE.md](REFERENCE.md).
