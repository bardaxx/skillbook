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
