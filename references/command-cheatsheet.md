## Core `agy` subcommands

- Show help and changelog:
  `agy help`
  `agy changelog`
- Install the CLI (one-time, also runs on first `agy` invocation):
  `curl -fsSL https://antigravity.google/cli/install.sh | bash`
  `agy install [--dir <path>] [--skip-aliases] [--skip-path]`
- List available models (auth required):
  `agy models`
- Manage plugins:
  `agy plugin add <name>`
  `agy plugins list`
- Self-update:
  `agy update`

## Global flags

- Add a workspace directory:
  `agy --add-dir <path>`
- Resume the most recent conversation in the current workspace:
  `agy -c`
  `agy --continue`
- Resume a specific conversation by id:
  `agy --conversation <id>`
- Auto-approve all tool permission requests (DANGEROUS — require explicit user consent per task):
  `agy --dangerously-skip-permissions`
- Run a one-shot prompt non-interactively:
  `agy -p "..."` / `agy --print "..."` / `agy --prompt "..."`
- Run a one-shot prompt in interactive mode (TUI):
  `agy -i "..."` / `agy --prompt-interactive "..."`
- Set the model:
  `agy --model <name>` / `agy -m <name>`
- Log to a file:
  `agy --log-file <path>`
- Override the print-mode timeout (default 5m0s):
  `agy --print-timeout <duration>`
- Enable sandbox mode:
  `agy --sandbox`

## TUI slash commands

- Authenticate / sign out:
  `/login`
  `/logout`
- Resume a conversation:
  `/resume`
- Switch model:
  `/model`
- Manage tool permission policy:
  `/permissions`
- Open the artifact review panel (also `ctrl+r`):
  `/review`
- Add a workspace directory, open a project:
  `/add-dir`
  `/open`
- Show changelog:
  `/changelog`
- Open settings:
  `/settings` (alias `/config`)
- Show pending diffs:
  `/diff`
- Memory, MCP, hooks, skills, tools, credits, help:
  `/memory`
  `/mcp`
  `/hooks`
  `/skills`
  `/tools`
  `/credits`
  `/help`
