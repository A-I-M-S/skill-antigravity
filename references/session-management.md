## Session rules

- `agy` keeps a per-workspace history in `~/.gemini/antigravity-cli/cache/projects.json`.
- The same workspace must always use the same conversation. Reusing conversations preserves context, decisions, and prior proposals.
- Start a new conversation **only** with explicit user approval.
- The agent never opens a new conversation without asking.

## Detecting an existing session

Read the cache and look up the current workspace:

```bash
WORKSPACE_KEY="$(realpath "$WORKSPACE")"
jq -r --arg k "$WORKSPACE_KEY" '.workspaces[$k] // empty' \
  ~/.gemini/antigravity-cli/cache/projects.json
```

If an entry exists, it lists the conversation ids associated with that workspace (typically the most recent first).

The agent must surface the discovered conversation id (or "no existing conversation") to the user and ask whether to resume or start fresh. When in doubt, ask.

## Resuming a session

`agy` exposes two resume flags that work in both `-p` and TUI modes:

```bash
# Resume the most recent conversation in the current workspace
agy -c -p "<prompt>" --model "Gemini 3.1 Pro (High)" --sandbox

# Resume a specific conversation by id
agy --conversation "<id>" -p "<prompt>" --model "Gemini 3.1 Pro (High)" --sandbox
```

`--continue` is the long form of `-c`. `--conversation <id>` takes a specific id from the cache.

Verify a conversation exists before resuming; `agy --conversation <bogus>` fails fast. If the id is wrong, fall back to `-c` or start a new conversation only with user approval.

## The Plan → approve → execute loop, in session terms

The three `agy` invocations for a single task share one conversation:

1. **Plan** — first invocation, no `-c`. This **creates** the conversation.
2. **Revise** (optional, multiple rounds) — `agy -c -p "<revision prompt>"` — picks up the same conversation and re-emits the proposal.
3. **Build** — `agy -c -p "<build prompt>" --dangerously-skip-permissions` — picks up the same conversation and executes the approved plan.

If the user starts a new task in the same workspace, the agent must decide: continue the existing conversation (if the new task is related), or start a new one (if it's unrelated). The decision is the user's, not the agent's.

## Starting a new session

Only with explicit user approval. The agent must say, in chat, "Starting a new conversation in `<workspace>`" and confirm the workspace path before invoking `agy` without `-c` or `--conversation`.

A new conversation is created by running `agy -p "..."` (no `-c`, no `--conversation`) from the workspace. The CLI auto-creates the entry in `projects.json`.

## When unsure whether to resume or start fresh

Ask the user. The cost of one extra confirmation is much lower than losing context or accidentally creating parallel conversations that drift.

## What does **not** exist

- There is no `agy session` subcommand. Use `-c` / `--continue` / `--conversation <id>`.
- There is no `/resume` slash command in the TUI that the agent drives (the TUI exposes a conversation picker on launch, but the agent does not drive the TUI).
- There is no `--new` flag. A new conversation is implicit when you do not pass `-c` or `--conversation`.
