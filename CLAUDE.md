# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

This repository is a **skills collection** built using the [Agent Skills](https://agentskills.io)
open format. It contains skills for AI agents (Claude Code, VS Code Copilot, etc.) that help
business users analyze requirements and generate test artifacts.

This is **not a code project** — it is a prompt-engineering and content project. The primary
artifacts are `SKILL.md` files under `skills/`.

## Repository structure

```
.claude-plugin/
└── marketplace.json          # Plugin manifest listing all skills
skills/
└── wlb-test-engineer/
    ├── SKILL.md              # Skill metadata + instructions
    ├── reference.md          # Test design techniques (BVA, equivalence partitioning, etc.)
    └── test-case-template.md # Reusable output template
```

## Agent Skills format

- Each skill is a directory under `skills/` containing a `SKILL.md` with YAML frontmatter (`name`, `description`) and markdown instructions.
- The directory name **must match** the `name` field in frontmatter.
- `name` must be lowercase alphanumeric + hyphens, max 64 chars, no leading/trailing/consecutive hyphens.
- `description` should explain both what the skill does and when to activate it (max 1024 chars).
- Keep `SKILL.md` under 500 lines / ~5000 tokens. Move detailed content to separate reference files.
- Full spec: https://agentskills.io/specification

## Adding a new skill

1. Create `skills/<skill-name>/SKILL.md` with proper frontmatter
2. Add the skill entry to `.claude-plugin/marketplace.json`
3. Update `README.md` and `CHANGELOG.md`

## How to validate

```bash
skills-ref validate ./skills/wlb-test-engineer
```

## Key editing guidelines

- When editing `SKILL.md`, preserve the YAML frontmatter format exactly.
- Skills target **business users** — language must stay non-technical and approachable.
- Test artifacts use a consistent ID scheme: scenarios as `TS-XX`, test cases as `TC-XX`.
- Skills support requirements provided in any language and should respond in the same language.
