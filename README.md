# Skillbook

**Portable playbooks for coding agents** — markdown workflows any coding agent can load, version, and share across repositories.

This catalog is separate from application code. You install skills into a consumer repo; the repo’s own `AGENTS.md` (or equivalent) tells agents when to use them. Layout rules and glossary live in [AGENTS.md](AGENTS.md) and [CONTEXT.md](CONTEXT.md).

## Why this exists

Agents work better with small, explicit procedures than with one-off prompts. Each skill here is a folder with a `SKILL.md` (and optional `REFERENCE.md` / `README.md`) that describes **when** to act and **what** to do — orchestration, not a framework.

Shipped skills live under `skills/engineering/`. Full bucket layout (productivity, misc, drafts, and so on) is defined in [AGENTS.md](AGENTS.md).

## Skills

| Skill | What it does |
|-------|----------------|
| [**openspec-program**](skills/engineering/openspec-program/SKILL.md) | Split a large PRD or epic into prioritized OpenSpec **slices**, track them in a program register, and keep status in sync with propose → apply → archive. |

Human-oriented overview: [openspec-program/README.md](skills/engineering/openspec-program/README.md).

```
PRD / epic  →  program register (slices)  →  one OpenSpec change per slice
```

## Use in your repository

1. Copy or install the skill folder into your agent skills path (e.g. `.agents/skills/<name>/`).
2. Document it in your repository **agent instructions** (`AGENTS.md` or mirrors) — especially for [openspec-program](skills/engineering/openspec-program/SKILL.md), which expects a program layer in the consumer repo.
3. Invoke by skill name when the task matches the skill description.

Some skills assume other tooling in the consumer repo (e.g. [OpenSpec](https://github.com/Fission-AI/OpenSpec), PRD workflows). They are referenced, not bundled here.

## Contributing

Adding or shipping a skill: follow [AGENTS.md](AGENTS.md) (bucket, bucket `README.md`, catalog entry, `.claude-plugin/plugin.json`).

## License

[MIT](LICENSE) — Copyright (c) 2026 Bardaxx.
