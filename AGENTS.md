# Repository Guidelines

## Project Structure & Module Organization
- Root: `apple_mail_mcp.py` hosts FastMCP tools and AppleScript helpers; `start_mcp.sh` bootstraps a venv and runs the server; `requirements.txt` tracks Python deps (FastMCP).
- Bundling: `apple-mail-mcpb/` contains `build-mcpb.sh` and `manifest.json` to produce `.mcpb` bundles; outputs land in the repo root.
- Skill content: `skill-email-management/` holds the Claude Code skill (`SKILL.md`, examples, templates) plus a distributable archive `email-management-skill.zip`.
- Config samples: `claude_desktop_config_example.json` shows how to register the server in Claude Desktop.
- Docs/changelog: `README.md`, `CHANGELOG.md`, `CLAUDE.md`.

## Build, Test, and Development Commands
- Environment: `python3 -m venv venv && source venv/bin/activate && pip install -r requirements.txt` (Python 3.7+).
- Run server locally: `./start_mcp.sh` (creates/updates `venv/` and executes `apple_mail_mcp.py`); for direct runs inside an active venv: `python apple_mail_mcp.py`.
- Bundle for distribution: `cd apple-mail-mcpb && ./build-mcpb.sh` → `../apple-mail-mcp-v*.mcpb`.
- Install skill to user scope: `cp -r skill-email-management ~/.claude/skills/email-management`.

## Coding Style & Naming Conventions
- Python: follow PEP 8 (4-space indent, snake_case for functions/vars, UpperCamelCase for classes); prefer explicit imports and type hints.
- Tool docstrings power the MCP UI—keep them crisp, imperative, and user-facing; note safety limits and parameters.
- AppleScript strings live inline; keep them self-contained, defensive (try/else), and timeout-aware as in `run_applescript`.
- Files are ASCII; avoid introducing non-ASCII unless mirroring existing content.

## Testing Guidelines
- No automated test suite yet; smoke-test changes by running `./start_mcp.sh` and exercising tools via an MCP client (Claude Desktop) or calling functions directly within a venv.
- Validate high-risk flows (move/delete/status updates) against a non-production mailbox first; confirm safety limits (`max_moves`, `max_deletes`, etc.) remain intact.
- For bundle changes, build with `build-mcpb.sh`, then install the `.mcpb` in Claude Desktop and run a basic inbox overview + search to confirm wiring.

## Commit & Pull Request Guidelines
- Commit messages follow the existing short, imperative style (`Add ...`, `Fix ...`, `Update ...`); keep scope tight.
- PRs should summarize intent, list manual test notes, and call out Mail permissions or configuration steps when relevant.
- Update docs when behavior or safety defaults change (`README.md`, `AGENTS.md`, bundle `README` if altered); bump `CHANGELOG.md` when releasing.

## Security & Configuration Tips
- Apple Mail access requires macOS automation permissions; document any new prompts or scopes when you alter AppleScript behavior.
- Avoid storing personal mail data; keep samples generic. Respect safety limits by default, and surface opt-in parameters explicitly.
