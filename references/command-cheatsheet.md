## Real `agy` surface (v1.0.6)

This is the complete CLI surface, validated against the binary. Anything not listed here either does not exist or is not safe to assume.

## Global flags

```text
--add-dir <path>                 Add a directory to the workspace (repeatable)
-c, --continue                   Resume the most recent conversation in the current workspace
--conversation <id>              Resume a specific conversation by id
--dangerously-skip-permissions   Auto-approve all tool permission requests (DANGEROUS)
-i, --prompt-interactive         Run an initial prompt interactively, then continue in TUI
--log-file <path>                Override CLI log file path
--model <name>                   Set the model for this invocation (exact string from `agy models`)
-p, --print, --prompt            Run a single prompt non-interactively; response on stdout
--print-timeout <duration>       Print-mode wait timeout (default 5m0s)
--sandbox                        Run with terminal sandbox restrictions enabled
```

`--print-timeout` accepts Go duration syntax (`5m`, `90s`, `1h30m`). Default is 5 minutes — long enough for most plan-and-build cycles, may need bumping for heavy refactors.

## Subcommands

```text
agy changelog       Show changelog and release notes
agy help [subcmd]    Show help for a subcommand
agy install         Configure environment paths and shell settings
agy models          List available models (auth required)
agy plugin ...      Manage plugins (list, import, install, uninstall, enable, disable, validate, link)
agy plugins         Alias for `agy plugin`
agy update          Update the CLI binary
```

- `agy install` is for PATH/shell setup (flags: `--dir`, `--skip-aliases`, `--skip-path`). It is **not** the auth command.
- `agy plugin` exposes: `list`, `import [source]`, `install <target>` (supports `plugin@marketplace`), `uninstall <name>`, `enable <name>`, `disable <name>`, `validate [path]`, `link <marketplace> <target>`.

## Real TUI slash commands (TUI-only — not for `-p`)

The agent does not drive the TUI. These are listed for completeness, since the user may run them when interacting with `agy` directly.

```text
/help          Show help
/changelog     Show changelog
/logout        Clear the OAuth token
/settings      Open settings panel (alias: /config)
/model         Pick the model for this TUI session
/permissions   Add/edit/remove tool-permission rules
```

- `/permissions` was added in v1.0.5 — it lets the user manage permission rules for tool calls directly in the CLI. TUI-only.
- TUI slash commands do **not** work in `-p` mode. The agent must not try to send them through stdin.

## Flags that are commonly confused or that **do not exist**

The skill does not use these because they are not in the real CLI:

- `--plan` — does not exist. Planning is forced by prompt phrasing.
- `--mode` — does not exist. There is no Fast/Planning mode toggle.
- `--login`, `/login` — does not exist. Auth is via the browser OAuth flow on first run.
- `--session`, `agy session` — does not exist. Use `--continue` / `-c` / `--conversation <id>`.
- `--config`, `agy config` — does not exist. Config is in `settings.json`.
- `--mcp`, `agy mcp` — does not exist. MCP servers are configured in `mcp_config.json`.
- `--tools`, `--agents`, `--init` — do not exist.
- `/resume`, `/review`, `/memory`, `/mcp`, `/hooks`, `/skills`, `/tools`, `/credits`, `/add-dir`, `/open`, `/diff`, `/config` — most of these are TUI slash commands that may exist in some form, but the skill does not depend on them. Verify with `agy help` if needed.

## Useful one-liners

```bash
# Run a planning prompt headless
agy -p "Plan: ..." --model "Gemini 3.1 Pro (High)" --sandbox --add-dir "$PWD"

# Resume and build
agy -p "Build the approved plan" -c --model "Gemini 3.1 Pro (High)" --sandbox --dangerously-skip-permissions

# List models
agy models

# Verify auth
agy models && echo "authed" || echo "not authed"

# Show current default model + project
cat ~/.gemini/antigravity-cli/settings.json

# Find a conversation id for the current workspace
jq '.workspaces["'"$PWD"'"]' ~/.gemini/antigravity-cli/cache/projects.json

# Tail the most recent log
tail -f "$(ls -t ~/.gemini/antigravity-cli/log/*.log | head -1)"
```

## Version check

```bash
agy --version
```

The skill is validated against v1.0.6. If a user has an older or newer version, run `agy changelog` to learn what changed; the operator-prompt pattern still works because it depends only on `-p`, `-c`, `--model`, `--sandbox`, and `--dangerously-skip-permissions`, all of which have been stable since v1.0.5.
