# WLB Skills Collection

A collection of [Agent Skills](https://agentskills.io) that help business users, QA analysts, and developers turn requirements into comprehensive test artifacts.

## Skills

| Skill | Description |
|-------|-------------|
| [wlb-test-engineer](skills/wlb-test-engineer/) | Analyze business requirements and generate test scenarios, test cases, and test data |

## Installation

Copy or symlink the skill directories to your agent's skill discovery path:

### Claude Code

```bash
# Project-level
cp -r skills/wlb-test-engineer .agents/skills/wlb-test-engineer

# Global
cp -r skills/wlb-test-engineer ~/.claude/skills/wlb-test-engineer
```

### VS Code Copilot (Agent mode)

```bash
cp -r skills/wlb-test-engineer .agents/skills/wlb-test-engineer
```

## Usage

Once installed, ask your agent something like:

- "Analyze this requirement and generate test cases"
- "What test scenarios should we cover for this feature?"
- "Generate test data for this user story"
- "Review my acceptance criteria for testability"

## Adding a new skill

1. Create a new directory under `skills/` with a `SKILL.md` file
2. The directory name must match the `name` field in `SKILL.md` frontmatter
3. Add the skill entry to `.claude-plugin/marketplace.json`
4. Update this README

## Format Reference

Skills follow the [Agent Skills specification](https://agentskills.io/specification). Each skill directory must contain at minimum a `SKILL.md` with YAML frontmatter (`name` and `description` required).
