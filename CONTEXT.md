# Skillbook

Portable playbooks for coding agents: markdown instructions agents load to run repeatable workflows. This repository owns the skills, their buckets, and the catalog rules—not the application code of repos that install them.

Domain terms below apply **here**. Consumer repos (applications) may have their own `CONTEXT.md` for product language; do not merge the two.

## Language

### Catalog and layout

**Skill**  
A folder with a `SKILL.md` (required) plus optional `README.md`, `REFERENCE.md`, or scripts. The unit agents invoke by name or path.  
_Avoid_: command, plugin, rule (unless quoting a specific tool’s UI)

**Bucket**  
A category folder under `skills/` that groups skills by intent and visibility (`engineering/`, `productivity/`, `misc/`, `personal/`, `in-progress/`, `deprecated/`).  
_Avoid_: category tag, namespace (unless describing paths only)

**Shipped skill**  
A skill in `engineering/`, `productivity/`, or `misc/` that is listed in the top-level `README.md` and `.claude-plugin/plugin.json`.  
_Avoid_: published, production skill

**Catalog**  
The pair of index files (`README.md` + `plugin.json`) that advertise shipped skills to humans and tooling.  
_Avoid_: registry (unless referring to an external package registry)

**Unshipped skill**  
A skill in `personal/`, `in-progress/`, or `deprecated/`. It must not appear in the catalog.  
_Avoid_: private skill, draft (use bucket name instead)

**Consumer repo**  
Any repository that installs or copies skills from this catalog into its own agent layout (e.g. `.agents/skills/`, a plugin bundle). Consumer rules live in that repo’s agent instruction files, not here.  
_Avoid_: target repo, client project

### OpenSpec roadmap (cross-repo workflow)

**Roadmap**  
The single planning file `openspec/roadmap.md` in a consumer repo, listing slices, priorities, and lifecycle status. It is not an OpenSpec change.  
_Avoid_: backlog file, multiple parallel roadmap files (not supported yet)

**Slice**  
One decomposed unit of work inside the roadmap, with an id (`F01`, `R02`, `T03`) and a candidate OpenSpec change id.  
_Avoid_: ticket, story (unless the consumer’s issue tracker uses those terms externally)

**Slice prefix**  
`F` (product feature), `R` (refactoring), `T` (testing/quality), `D` (documentation), `I` (infrastructure/tooling, optional). Part of the slice id, not a separate entity.  
_Avoid_: type label, workstream

**Slice status**  
Lifecycle on a slice: `Ready`, `Spec Proposed`, `Applying`, `Applied`, `Archived`, or `Blocked`. Distinct from issue labels or git branch state.  
_Avoid_: stage, phase (generic)

**OpenSpec change**  
The artifact set under `openspec/changes/<change-id>/` in a consumer repo (proposal, design, tasks, etc.). Created by OpenSpec tooling/skills, not by the roadmap.  
_Avoid_: spec folder, change package

**OpenSpec CLI skills**  
Upstream skills such as `openspec-propose`, `openspec-apply-change`, and `openspec-archive-change`. `openspec-roadmap` orchestrates them; it does not replace them.  
_Avoid_: OpenSpec commands (ambiguous with the CLI binary)

### Agent configuration

**Repository agent instructions**  
Files in a consumer repo that tell every agent session how to work (commonly `AGENTS.md`, sometimes mirrored elsewhere). The roadmap layer must be documented there when `openspec-roadmap` is adopted.  
_Avoid_: system prompt, CLAUDE.md-only setup (too tool-specific)

## Relationships

- This repository contains many **skills** grouped into **buckets**.
- **Shipped skills** appear in the **catalog**; **unshipped skills** do not.
- A **consumer repo** may install one or more skills and optionally maintain a **roadmap** at `openspec/roadmap.md`.
- The **roadmap** contains many **slices**; each implementable **slice** maps 1:1 to an **OpenSpec change** (except documented audit-only cases).
- **Slice status** advances as **OpenSpec CLI skills** run; the roadmap is updated after each step.
- **Repository agent instructions** in the consumer repo point agents at installed skill paths and workflow rules.

## External dependencies (not defined here)

Skills in this catalog may **reference** workflows from other MIT-licensed collections (e.g. OpenSpec) without vendoring them. Install those skills separately in the consumer repo. See [README.md](README.md).

## Flagged ambiguities

| Term | Resolution |
|------|------------|
| `TIMELINE_*` / `openspec/programs/` | Obsolete planning paths. **Merge** into **`openspec/roadmap.md`**, then **delete** legacy files (do not rename in place); see `openspec-roadmap` REFERENCE legacy migration. |
| “change” | In consumer repos, prefer **OpenSpec change**. In this repo, prefer **skill** for catalog items. |
| “PRD” | Product/requirements document in the consumer’s **issue tracker** or docs—not a file type owned by Skillbook. |
| “plugin” | May mean `.claude-plugin/plugin.json` (catalog) or a host’s plugin system; specify which. |
