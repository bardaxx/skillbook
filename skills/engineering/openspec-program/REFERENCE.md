# OpenSpec Timeline Register — Reference

Use this file when bootstrapping `openspec/TIMELINE_<context>.md` or adding slices.

---

## `openspec/config.yaml` baseline

Use this baseline for token and loading limits.
Treat this config as operational guidance only: do not regenerate, rewrite, or recreate OpenSpec specs from `openspec/config.yaml`.

```yaml
openspec_program:
  mode: default
  timeline:
    filename_pattern: "TIMELINE_<context>.md"
    path: "openspec/"
  token_budget:
    max_input_tokens: 18000
    reserved_output_tokens: 4000
  context_loading:
    include_only:
      - active_timeline_register
      - selected_slice
      - linked_openspec_change
      - directly_referenced_files
    load_full_prd_when: "slice_is_ambiguous"
  pruning:
    compact_history_after_archived_slices: 8
  quality_gate:
    applied_requires:
      - tests_pass
      - affected_specs_updated
      - no_unrelated_files_changed
      - acceptance_verified
```

---

## Timeline template (default)

This is the default and preferred template. Keep it concise.
For a copy-ready file, use [TIMELINE_SKELETON.md](TIMELINE_SKELETON.md).

```markdown
# TIMELINE_<context>

PRD: <issue URL, path, or "see conversation YYYY-MM-DD">
This timeline is a short execution map used to generate OpenSpec changes per slice.
Keep entries concise. Do not duplicate proposal/design/tasks content.

## How To Use This Timeline

1. Pick the highest-priority `Ready` slice.
2. Create/update one OpenSpec change for that slice.
3. Move lifecycle forward and update the progress log.
4. Keep scope limited to that slice.

## Status Model

`Ready` -> `Spec Proposed` -> `Applying` -> `Applied` -> `Archived` (plus `Blocked`)

## Slices

### T01 - <short title>
Status: `Ready`
Goal: <1-2 lines>
Candidate OpenSpec change id: `<kebab-case-change-id>`
Spec link: `openspec/changes/<change-id>/`
Files:
- `path/to/file.ts`
Notes: <one short line or "none">
Progress:
- Proposed: pending
- Applying: pending
- Applied: pending
- Archived: pending

## Dependencies

### T01
Depends on: none
Blocks: T02
Can run in parallel: no

## Recommended Execution Order

1. T01 - <short title>
2. T02 - <short title>

## Compacted history

Keep only short archived summaries after the configured threshold.

- <slice-id> -> <outcome> -> <changed files> -> <validation>

## Post-implementation reality check

For every `Applied` slice, append:

- What changed from original plan:
- Unexpected issues:
- Follow-up needed:
```

---

## Slice field reference

| Field | Required | Notes |
|-------|----------|-------|
| Status | Yes | One lifecycle value |
| Goal | Yes | One short outcome statement |
| Candidate OpenSpec change id | Yes | kebab-case; 1:1 with change dir |
| Spec link | Yes | Path to the OpenSpec change dir |
| Files | Yes | 1-5 starting files |
| Notes | Optional | Keep to one short line |
| Progress | Yes | Update at each lifecycle step |

Do not add full-spec style sections by default.

---

## Definition of done for `Applied`

A slice cannot be marked `Applied` unless:

- tests pass
- affected docs/specs are updated
- no unrelated files changed
- spec acceptance criteria are verified

---

## Do-not-use gate

Do not use the timeline flow for:

- bugfixes under 30 minutes
- isolated copy/UI changes
- one-file refactors
- exploratory spikes

---

## Scope changes in timeline

### Add a new in-flight feature

- Add a new `### <ID> - <Title>` block with default timeline fields.
- Default status is `Ready` unless implementation already started.
- Insert item in `## Recommended Execution Order` with a short reason.

### Deprecate instead of delete

- Keep the slice block in place.
- Add `Deprecated on YYYY-MM-DD: <reason>` in `Notes`.
- Remove from active `Recommended Execution Order`.
- Add a compacted-history entry if needed.

### Restore a deprecated slice

- Add `Restored on YYYY-MM-DD` note.
- Set status back to actionable value (usually `Ready`).
- Reinsert into `Recommended Execution Order`.

Hard deletion is allowed only when explicitly requested by the user.

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
- [ ] `openspec/config.yaml` contains mode/token/loading limits
- [ ] timeline file is named `openspec/TIMELINE_<context>.md`
- [ ] PRD/issue link at top
- [ ] How-to + status model sections
- [ ] Lite slices only (concise fields)
- [ ] Candidate change ids are unique kebab-case
- [ ] Dependencies map filled for active slices
- [ ] Recommended execution order
- [ ] Compacted history section present
- [ ] Post-implementation reality check section present
```

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
| 2 | Decompose into prioritized slices | `openspec/TIMELINE_<context>.md` |
| 3 | Per slice with status `Ready` | `openspec-propose` → `openspec/changes/<change-id>/` |
| 4 | Implement | `openspec-apply-change` (slice → `Applying` → `Applied`) |
| 5 | Complete | `openspec-archive-change` (slice → `Archived`) |

**Rules**

- Program registers are planning files only — do not copy `proposal.md` / `design.md` / `tasks.md` into them.
- Implementable slices are **1:1** with OpenSpec changes (`Candidate OpenSpec change id`).
- Slice lifecycle: `Ready` → `Spec Proposed` → `Applying` → `Applied` → `Archived` (`Blocked` when decisions are pending).
- Pick the next slice by priority and recommended execution order in the register.
- Update the register after every propose, apply, and archive step.
- Keep timeline entries concise and operational.
- Enforce context/token limits via `openspec/config.yaml`.

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

- `<context>` — `openspec/TIMELINE_<context>.md` (PRD: <link>)
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
