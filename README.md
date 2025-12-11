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

### Via Claude Code Plugin System

```bash
# 1. Add the marketplace (from local path)
/plugin marketplace add /path/to/appsentinels-skills

# 2. Install the skill
/plugin install appsentinels@appsentinels-skills
```

### From GitHub

```bash
# 1. Clone the repository
git clone https://github.com/vinay4appsentinels/appsentinels-skills.git

# 2. Add the marketplace
/plugin marketplace add /path/to/appsentinels-skills

# 3. Install the skill
/plugin install appsentinels@appsentinels-skills
```

### Verify Installation

After installation, restart Claude Code and verify:
- The skill appears in `/plugin list`
- Plugin cache exists at `~/.claude/plugins/cache/appsentinels-skills/`

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

## Prerequisites

### Install as-mcp-cli

```bash
# From source
git clone <as-mcp-cli-repo>
cd as-mcp-cli
pip install --user .

# Or via pip (if published)
pip install as-mcp-cli
```

### Configure MCP Server

```bash
# 1. Add the AppSentinels MCP server (opens browser for OAuth)
as-mcp-cli add appsentinels https://mcp.appsentinels.ai/mcp/sse

# 2. Verify setup
as-mcp-cli list                                    # Show configured servers
as-mcp-cli mcp appsentinels tenant all-tenants     # Test connection

# 3. Re-authenticate if token expires
as-mcp-cli auth appsentinels --force
```

**Credentials**: Stored in `~/.claude/.credentials.json`

See [references/installation.md](skills/appsentinels/references/installation.md) for detailed setup instructions.

## License

MIT
