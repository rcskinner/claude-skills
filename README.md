# claude-skills

A collection of Claude Code plugins for the `rcskinner-skills` marketplace.

## Plugins

| Plugin | Description |
|--------|-------------|
| `test` | Smoke-test plugin — validates that the marketplace is wired up correctly |
| `code-critic` | Focused, opinionated code review skill |

## Installation

### 1. Register the marketplace

Add the following to `~/.claude/settings.json`:

```json
{
  "extraKnownMarketplaces": {
    "rcskinner-skills": {
      "source": {
        "source": "git",
        "url": "https://github.com/rcskinner/claude-skills.git"
      },
      "autoUpdate": true
    }
  }
}
```

### 2. Enable a plugin

Add the plugin to `enabledPlugins` in the same file. For example, to enable the `test` plugin:

```json
{
  "enabledPlugins": {
    "test@rcskinner-skills": true
  }
}
```

A full `settings.json` with both plugins enabled looks like this:

```json
{
  "enabledPlugins": {
    "test@rcskinner-skills": true,
    "code-critic@rcskinner-skills": true
  },
  "extraKnownMarketplaces": {
    "rcskinner-skills": {
      "source": {
        "source": "git",
        "url": "https://github.com/rcskinner/claude-skills.git"
      },
      "autoUpdate": true
    }
  }
}
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
