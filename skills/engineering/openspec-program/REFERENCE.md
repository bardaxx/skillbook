# OpenSpec Timeline Register — Reference

Use this file when bootstrapping `openspec/TIMELINE_<context>.md` or adding slices.
All guidance in this reference must stay token-saving and operationally concise.

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

The default template is maintained only in [TIMELINE_SKELETON.md](TIMELINE_SKELETON.md).
Do not duplicate or rewrite that template in this reference.
When needed, link to it and keep this document focused on rules, constraints, and checklists.

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

### Slice ID prefixes

| Prefix | Use |
|-------|-----|
| `F` | Product feature slices |
| `R` | Refactoring slices (no intended behavior change) |
| `T` | Testing and quality slices |
| `D` | Documentation-only slices |
| `I` | Infrastructure/tooling slices (optional) |

Use sequential numbering within each prefix (`F01`, `R02`, ...).

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
- Insert item in `## Recommended Execution Order` at the best position, not automatically at the end.
- Choose insertion position using:
  - dependency readiness (must come after required prerequisites)
  - risk reduction and unblock potential
  - expected leverage for upcoming slices
  - user urgency constraints, if provided
- Record a short rationale near the updated execution order (for example: "Inserted F05 after R03 to reuse new interfaces and reduce rework before F06").
- If insertion changes what should run next, explicitly state the new next slice.

### Add-next for immediate placement

- Use `add-next <slice-id> "<title>"` when the user explicitly wants the new slice as next work.
- Insert the new slice at the nearest valid next position.
- If dependencies block immediate placement, place at the earliest valid slot and explain the blocker.
- Record a short rationale near the updated execution order.

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
