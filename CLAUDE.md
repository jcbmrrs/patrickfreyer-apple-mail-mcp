# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Apple Mail MCP Server is a Model Context Protocol (MCP) server that provides AI assistants with natural language access to Apple Mail on macOS. Built with [FastMCP](https://github.com/jlowin/fastmcp), it includes:

1. **MCP Server** (`apple_mail_mcp.py`): 18 email management tools via AppleScript automation
2. **Email Management Expert Skill** (`skill-email-management/`): Claude Code skill with productivity workflows
3. **MCP Bundle** (`apple-mail-mcpb/`): Distributable `.mcpb` package for easy installation

## Development Commands

### Testing the MCP Server

Run the server directly for testing:
```bash
# Activate virtual environment
python3 -m venv venv
source venv/bin/activate

# Install dependencies
pip install -r requirements.txt

# Run the server (will start in MCP mode)
python3 apple_mail_mcp.py
```

### Building the MCP Bundle

Create a distributable `.mcpb` package:
```bash
cd apple-mail-mcpb
./build-mcpb.sh
```

This creates `apple-mail-mcp-v{version}.mcpb` in the repository root. The build script:
- Reads version from `apple-mail-mcpb/manifest.json`
- Bundles server code, requirements, startup script, and skill
- Creates a zip with `.mcpb` extension

### Installing the Email Management Skill

The skill is installed separately to Claude Code's user scope:
```bash
cp -r skill-email-management ~/.claude/skills/email-management
```

### Testing with Claude Desktop

Manual installation for testing (edit `~/Library/Application Support/Claude/claude_desktop_config.json`):
```json
{
  "mcpServers": {
    "apple-mail": {
      "command": "/path/to/repo/venv/bin/python3",
      "args": ["/path/to/repo/apple_mail_mcp.py"],
      "env": {
        "USER_EMAIL_PREFERENCES": "Your preferences here"
      }
    }
  }
}
```

Then restart Claude Desktop.

## Architecture

### MCP Server (`apple_mail_mcp.py`)

**Core Pattern**: FastMCP tools → AppleScript execution → Apple Mail automation

**Key Components**:
- `run_applescript()`: Executes AppleScript with 120s timeout
- `parse_email_list()`: Parses structured AppleScript output into Python dicts
- `@inject_preferences`: Decorator that appends `USER_EMAIL_PREFERENCES` env var to tool docstrings

**Tool Structure**: Each tool follows this pattern:
1. Decorated with `@mcp.tool()` and `@inject_preferences`
2. Builds AppleScript string with parameters
3. Calls `run_applescript()` for execution
4. Parses and formats results

**Safety Limits**: Tools include safety parameters to prevent accidental bulk operations:
- `update_email_status`: max_updates (default 10)
- `manage_trash`: max_deletes (default 5)
- `move_email`: max_moves (default 1)

### Email Management Skill (`skill-email-management/`)

**Purpose**: Teaches Claude intelligent email workflows and best practices, transforming raw MCP tools into expert assistance.

**Structure**:
- `SKILL.md`: Core workflows, tool orchestration patterns, safety principles
- `examples/`: Detailed workflow guides (Inbox Zero, triage, folder organization)
- `templates/`: Copy-paste patterns (common workflows, search patterns)

**Activation**: Triggers automatically when Claude Code detects email management keywords (inbox zero, triage, organize emails, etc.)

### MCP Bundle (`apple-mail-mcpb/`)

**Purpose**: Distributable package for one-click installation in Claude Desktop.

**Key Files**:
- `manifest.json`: MCP Bundle metadata (version, tools, user config schema)
- `build-mcpb.sh`: Builds the `.mcpb` zip package
- `start_mcp.sh`: Copied into bundle, creates venv on user's machine on first run

**Version Management**: Version is defined in `manifest.json` and used by build script.

### Startup Wrapper (`start_mcp.sh`)

**Purpose**: Ensures Python environment is set up correctly on user's machine.

**Behavior**:
1. Checks if `venv/` exists at runtime
2. If missing, creates venv with user's Python installation
3. Installs dependencies from `requirements.txt`
4. Executes `apple_mail_mcp.py` with the venv Python

This avoids bundling Python binaries and handles version conflicts across different macOS environments.

## AppleScript Integration

The server uses AppleScript to interact with Apple Mail. Key patterns:

**Email Queries**: AppleScript loops through Mail accounts/mailboxes and formats output with structured delimiters:
```applescript
set output to "✉ Subject line" & linefeed
set output to output & "From: sender@email.com" & linefeed
```

**Parsing**: `parse_email_list()` looks for indicators like `✉` (unread) or `✓` (read) and field prefixes like `From:`, `Date:`.

**Performance**: Content fetching (`content of message`) is slower than metadata. Tools include `include_content` parameters to optimize when preview isn't needed.

**Mailbox Paths**: Nested mailboxes use "/" separator (e.g., `"Projects/Amplify Impact"`). Exchange accounts may have different naming conventions.

## User Preferences System

**Mechanism**:
- User sets preferences in Claude Desktop MCP config (via UI or JSON)
- Preferences stored in `USER_EMAIL_PREFERENCES` environment variable
- `@inject_preferences` decorator appends to every tool's docstring
- Claude sees preferences as part of tool descriptions

**Purpose**: Helps Claude make workflow decisions aligned with user preferences (default account, folder preferences, result limits).

## Version Management

When releasing a new version:

1. Update version in `apple-mail-mcpb/manifest.json`
2. Update `CHANGELOG.md` with changes
3. Run `apple-mail-mcpb/build-mcpb.sh` to build new bundle
4. The script automatically reads version from manifest
5. Creates `apple-mail-mcp-v{version}.mcpb` in repo root

## Skill Development

The Email Management Skill demonstrates Claude Code skill best practices:

**Skill Metadata** (YAML frontmatter in `SKILL.md`):
```yaml
name: email-management-expert
description: Expert email management assistant... Use this when...
```

**Activation Keywords**: Listed in description trigger skill loading (inbox zero, triage, organize emails, etc.)

**Content Organization**:
- Core principles and tool orchestration in main SKILL.md
- Detailed workflows in examples/ (referenced from main skill)
- Reusable templates in templates/ (search patterns, common workflows)

**Skill Installation**: Skills are user-scoped (`~/.claude/skills/`) and activate across all projects when keywords match.

## Important Constraints

**macOS Only**: Requires Apple Mail and AppleScript, no cross-platform support possible.

**Permissions Required**:
- Mail.app Control (System Settings > Privacy & Security > Automation)
- Mail Data Access (for reading email content)

**Mail.app Must Run**: AppleScript automation requires Mail.app to be running.

**Script Timeout**: AppleScript execution has 120s timeout (see `run_applescript()`). Large mailbox operations may hit this limit.

## File Structure

```
apple-mail-mcp/
├── apple_mail_mcp.py              # Main MCP server (FastMCP)
├── requirements.txt               # Python deps (fastmcp>=0.1.0)
├── start_mcp.sh                   # Runtime venv setup wrapper
├── apple-mail-mcpb/
│   ├── manifest.json              # MCP Bundle metadata & version
│   └── build-mcpb.sh             # Bundle build script
└── skill-email-management/        # Claude Code skill
    ├── SKILL.md                   # Core workflows & orchestration
    ├── examples/                  # Detailed workflow guides
    └── templates/                 # Reusable patterns
```
