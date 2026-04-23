# claude-skills

Claude Code skills for developing, testing, and operating OpenShift.

## Installation

Add the marketplace in Claude Code:

```
/plugins marketplace add RomanBednar/claude-skills
```

Install the plugin:

```
/plugins install testing-skill@rb-claude-skills
```

## Updating

```
/plugins update testing-skill@rb-claude-skills
```

## Available Skills

| Skill | Description |
|-------|-------------|
| [test-plan](skills/test-plan/) | IEEE 829-inspired test plan authoring and execution |

## Adding a New Skill

1. Create `skills/<name>/SKILL.md` with frontmatter and instructions
2. Add the path to the `skills` array in `.claude-plugin/marketplace.json`
3. Bump the version and push
