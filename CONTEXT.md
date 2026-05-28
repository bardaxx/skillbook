# Skillbook

Portable playbooks for coding agents: markdown instructions agents load to run repeatable workflows. This repository owns the skills, their buckets, and the catalog rules—not the application code of repos that install them.

Domain terms below apply **here** (this catalog repository). Consumer repos (applications) may have their own `CONTEXT.md` for product language. Workflow-specific glossaries live with each skill (for example [openspec-program/REFERENCE.md](skills/engineering/openspec-program/REFERENCE.md#domain-terms)).

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

### Agent configuration

**Repository agent instructions**  
Files in a consumer repo that tell every agent session how to work (commonly `AGENTS.md`, sometimes mirrored elsewhere). The consumer documents which installed skills to use and when—not defined in Skillbook itself.  
_Avoid_: system prompt, tool-specific-only setup files as the sole source of truth

## Relationships

- This repository contains many **skills** grouped into **buckets**.
- **Shipped skills** appear in the **catalog**; **unshipped skills** do not.
- A **consumer repo** may install one or more skills from the catalog.
- **Repository agent instructions** in the consumer repo point agents at installed skill paths and workflow rules.

## External dependencies (not defined here)

Skills in this catalog may **reference** workflows from other MIT-licensed collections (e.g. OpenSpec) without vendoring them. Install those skills separately in the consumer repo. See [README.md](README.md).

## Flagged ambiguities

| Term | Resolution |
|------|------------|
| “change” | In this repo, prefer **skill** for catalog items. In consumer repos, “change” may mean something else (e.g. an OpenSpec change)—see that repo or the relevant skill glossary. |
| “plugin” | May mean `.claude-plugin/plugin.json` (catalog) or a host’s plugin system; specify which. |
