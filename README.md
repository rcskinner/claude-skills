# claude-skills

A collection of Claude Code plugins for the `rcskinner-skills` marketplace.

## Plugins

| Plugin | Description |
|--------|-------------|
| `test` | Smoke-test plugin — validates that the marketplace is wired up correctly |
| `code-critic` | Focused, opinionated code review skill |

## Installation

### 1. Register the marketplace

```bash
/plugin marketplace add https://github.com/rcskinner/claude-skills.git
```

### 2. Install a plugin

```bash
claude plugin install test@rcskinner-skills
```

### 3. Verify

Run `/hello` in Claude Code. You should see:

> Marketplace check: `rcskinner/claude-skills` is installed and working. The `test` plugin loaded successfully.

## Plugin structure

Each plugin lives in its own directory and follows this layout:

```
<plugin-name>/
  .claude-plugin/
    plugin.json       # name, version, description, skills path
  skills/
    <skill-name>/
      SKILL.md        # skill definition loaded by Claude Code
```
