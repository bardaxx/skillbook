# OpenSpec Program Register — Reference

Use this file when bootstrapping `openspec/programs/<slug>.md` or adding slices. Canonical legacy example: `openspec/TIMELINE_TEST.md`.

---

## Full register template

Copy and fill. Remove optional sections if unused.

```markdown
# <Program Title>

PRD: <issue URL, path, or "see conversation YYYY-MM-DD">

This file tracks work decomposed from the PRD into independently proposable OpenSpec
changes. It is a planning register, not an OpenSpec change.

Use this file when asking an agent to pick up the next slice. Choose the highest-priority
item marked `Ready`, create or update the matching OpenSpec change, implement only after
the OpenSpec gate is satisfied, and update this register at each lifecycle step.

## How To Use This Program

1. Pick the highest-priority item with status `Ready` (see Recommended Execution Order).
2. Create an OpenSpec change under `openspec/changes/` before implementation.
3. Use canonical domain terms from `CONTEXT.md`.
4. Run GitNexus exploration and impact analysis before modifying application symbols.
5. Update slice status when the change is proposed, applied, and archived.
6. Add links to the OpenSpec change, branch/PR, and tests as they become available.

Documentation-only edits to this file do not require a separate OpenSpec change.
Behavior, test, or production code changes do require OpenSpec.

## Status Model

| Status | Meaning | Required register update |
| --- | --- | --- |
| `Ready` | Understood enough to propose an OpenSpec change. | None. |
| `Spec Proposed` | Change exists under `openspec/changes/`. | Add change id/path. |
| `Applying` | Implementation started from approved change. | Branch/agent notes if useful. |
| `Applied` | Implementation and verification complete. | Test files + validation commands. |
| `Archived` | Change archived after completion. | Archive path + date. |
| `Blocked` | Needs product/domain/technical decision. | Blocking question + recommended answer. |

## Global Principles

<!-- Optional. Delete section if N/A. -->

- ...

## Context Snapshot

<!-- Optional: current state, known gaps, constraints relevant to all slices. -->

- ...

## Program

<!-- Repeat per slice. Group under ## Timeline or ## Feature slices etc. if helpful. -->

### T01 - <Short title>

Status: `Ready`

Priority: P0

Goal:
<One paragraph: what this slice delivers.>

Why it matters:
<Risk or value; tie to PRD.user stories where useful.>

Candidate OpenSpec change id:
`<kebab-case-change-id>`

Target behavior to specify:

- ...
- ...

Likely test type:
<Integration/E2E, unit, documentation-only, etc. — or "Likely verification" for non-test slices.>

Files to inspect:

- `path/to/file.ts`
- ...

Notes:
<Decisions deferred to OpenSpec, doc/code mismatches, etc.>

Open question:
<If any. Recommended answer on next line.>

Progress log:

- Proposed: pending
- Applying: pending
- Applied: pending
- Archived: pending

## Recommended Execution Order

1. T01 - ...
2. T02 - ...

The order may change if urgency shifts. When reordering, update this section and add a
short note explaining why.

## Agent Update Checklist

When proposing a spec:

- Change the item status to `Spec Proposed`.
- Add the OpenSpec change id/path.
- Add unresolved questions under the item, if any.

When applying a spec:

- Change the item status to `Applying`.
- Add branch or working context if useful.
- Keep scope tied to that slice unless the OpenSpec change says otherwise.

When implementation is done:

- Change the item status to `Applied`.
- Add test files created or changed.
- Add validation commands and results.
- Note production code changed to make tests pass.

When archiving:

- Change the item status to `Archived`.
- Add archive path and date.
- If follow-up work remains, create a new slice or update an existing one — do not
  hide follow-ups only in the progress log.
```

---

## Slice field reference

| Field | Required | Notes |
|-------|----------|-------|
| Status | Yes | One of the fixed lifecycle values |
| Priority | Yes | Default P0 / P1 / P2 |
| Goal | Yes | Outcome-oriented |
| Why it matters | Yes | Ties slice to PRD risk/value |
| Candidate OpenSpec change id | Yes | kebab-case; 1:1 with change dir |
| Target behavior to specify | Yes | Bullet list for propose input |
| Likely test type | Yes* | *Or "Likely verification" for audit/doc slices |
| Files to inspect | Yes | Starting points for propose/apply |
| Notes / Open question | If needed | Product or technical forks |
| Progress log | Yes | Update at each lifecycle step |

---

## Scope changes in timeline

Use these patterns when program scope changes after bootstrap.

### Add a new in-flight feature

- Add a new `### <ID> - <Title>` block with all minimum fields.
- Default status is `Ready` unless implementation is already underway and an OpenSpec change already exists.
- Insert the item in `## Recommended Execution Order` with a short note when added due to scope change.

### Deprecate instead of delete

When a slice is no longer planned, keep it for audit history:

- Keep the slice block in place (do not remove historical context).
- Add a clear marker in `Notes`, for example: `Deprecated on YYYY-MM-DD: <reason>`.
- Add a progress entry, for example: `- Deprecated: YYYY-MM-DD (<reason>)`.
- Remove the item from active `Recommended Execution Order` and note any replacement slice.

### Restore a deprecated slice

- Remove or supersede the deprecation note with a restore note.
- Set status back to an actionable value (usually `Ready`).
- Reinsert into `Recommended Execution Order`.

Hard deletion of a slice is allowed only when explicitly requested by the user.

---

## Progress log patterns

**After propose:**

```markdown
- Proposed: `openspec/changes/<change-id>/`
```

**After apply:**

```markdown
- Applying: `<branch or agent context>`
- Validation: `<command>` -> `<result>`
- Applied: `<key files changed>`
```

**After archive:**

```markdown
- Archived: `openspec/changes/archive/YYYY-MM-DD-<change-id>/` on `YYYY-MM-DD`
```

Use `pending` for steps not yet reached.

---

## Bootstrap checklist

```
- [ ] PRD/issue link at top
- [ ] How-to + status model sections
- [ ] Optional principles + context snapshot
- [ ] All slices with IDs (T/L/F) and minimum fields
- [ ] Candidate change ids are unique kebab-case
- [ ] Recommended execution order
- [ ] Agent update checklist (can copy from template)
- [ ] No OpenSpec artifact duplication
```

---

## Legacy path

`openspec/TIMELINE_TEST.md` uses the same register shape. New PRD-linked programs prefer `openspec/programs/<slug>.md`. Do not migrate legacy files unless the user asks.

---

## Agent docs snippet

Insert or merge into the repo’s primary agent instruction file (usually `AGENTS.md`), under or beside existing OpenSpec rules. Mirror the same block in any secondary entrypoints that load agent rules for that repository. Replace `<skill-path>` with the installed path.

### Section: OpenSpec Program (merge into OpenSpec workflow)

```markdown
## OpenSpec Program (PRD → multiple changes)

Use this layer when a PRD or epic would otherwise become one oversized OpenSpec change.

| Step | What | Where |
|------|------|--------|
| 1 | PRD or issue | Issue tracker / `PRD.md` file |
| 2 | Decompose into prioritized slices | `openspec/programs/<slug>.md` (legacy example: `openspec/TIMELINE_TEST.md`) |
| 3 | Per slice with status `Ready` | `openspec-propose` → `openspec/changes/<change-id>/` |
| 4 | Implement | `openspec-apply-change` (slice → `Applying` → `Applied`) |
| 5 | Complete | `openspec-archive-change` (slice → `Archived`) |

**Rules**

- Program registers are planning files only — do not copy `proposal.md` / `design.md` / `tasks.md` into them.
- Implementable slices are **1:1** with OpenSpec changes (`Candidate OpenSpec change id`).
- Slice lifecycle: `Ready` → `Spec Proposed` → `Applying` → `Applied` → `Archived` (`Blocked` when decisions are pending).
- Pick the next slice by priority and recommended execution order in the register.
- Update the register after every propose, apply, and archive step.

**Skill**

| Task | Skill file |
|------|------------|
| Bootstrap program, add slices, update lifecycle | `<skill-path>/SKILL.md` |
| Register template and checklists | `<skill-path>/REFERENCE.md` |
```

### Optional: active programs pointer

Add when bootstrapping or when the repo tracks multiple programs:

```markdown
### Active OpenSpec programs

- `<slug>` — `openspec/programs/<slug>.md` (PRD: <link>)
```

### Routing table variant (GitNexus-style repos)

If `AGENTS.md` already uses a task → skill file table, append:

```markdown
| Decompose PRD/epic into OpenSpec slices | `<skill-path>/SKILL.md` |
| Pick next program slice / update slice status | `<skill-path>/SKILL.md` |
```

### Agent docs checklist

```
- [ ] Primary agent instruction file contains OpenSpec Program subsection
- [ ] Secondary entrypoints mirrored where the repo uses them
- [ ] Skill path in table matches install location
- [ ] Active program link added when bootstrapping <slug>
- [ ] No duplicate conflicting OpenSpec instructions
```

---

## Doc-only / audit slices

For inventory or harness-audit work:

- Candidate id often `audit-<topic>` or `reclassify-<topic>`.
- Likely verification: documentation and planning; follow with per-cluster OpenSpec changes if needed.
- May remain a single change for the audit slice itself.
