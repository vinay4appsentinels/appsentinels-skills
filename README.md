# AppSentinels Skills for Claude Code

Claude Code skills for the AppSentinels API Security Platform.

## Skills

### appsentinels

API security platform CLI (`as-mcp-cli`) and ATDv2 threat detection rule authoring.

**Features:**
- as-mcp-cli command reference
- ATDv2 rule DSL syntax
- Rule creation workflow
- Coverage analysis

## Installation

### Via Claude Code

```bash
# Add as a plugin
claude plugins add appsentinels-skills
```

### Manual Installation

Copy the `skills/appsentinels/` folder to your Claude Code skills directory:

```bash
# Personal skills
cp -r skills/appsentinels ~/.claude/skills/

# Project skills
cp -r skills/appsentinels .claude/skills/
```

## Usage

Once installed, Claude will automatically invoke the skill when you:
- Work with `as-mcp-cli` commands
- Create ATDv2 threat detection rules
- Analyze API security
- Manage API tags

## Skill Contents

```
skills/appsentinels/
├── SKILL.md                    # Main skill file
└── references/
    ├── installation.md         # as-mcp-cli setup guide
    ├── cli-commands.md         # Full CLI reference
    ├── atdv2-dsl.md           # ATDv2 rule DSL syntax
    └── rule-workflow.md        # Rule creation workflow
```

## Requirements

- `as-mcp-cli` installed and configured
- AppSentinels MCP server access

## License

MIT
