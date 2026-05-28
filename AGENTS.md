# Agent instructions — Skillbook

Shared rules for any coding agent working in this repository. Domain terms: [CONTEXT.md](CONTEXT.md).

## Skills layout

Skills live under `skills/`.

## Catalog requirements (shipped skills)

Every shipped skill **must**:

1. Have an entry in the [top-level README.md](README.md), with the skill name linked to its `SKILL.md`.
2. Have an entry in [.claude-plugin/plugin.json](.claude-plugin/plugin.json).

Private, draft, or deprecated skills **must not** appear in either file.

## Adding or moving a skill

1. Add or move the skill folder under `skills/`.
2. Add or update the top-level `README.md`.
3. Add or update `.claude-plugin/plugin.json`.
4. Remove catalog entries if the skill is no longer shipped.
