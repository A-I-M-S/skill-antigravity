## Session rules

- `agy` keeps a per-workspace history in:
  `~/.gemini/antigravity-cli/cache/projects.json`
- The same workspace must always use the same conversation. Reusing conversations preserves context, decisions, and artifact history.
- Start a new conversation only with explicit user approval.

## Detecting an existing session

- Read `~/.gemini/antigravity-cli/cache/projects.json` and look up the current workspace key.
- If an entry exists, the file lists conversation ids associated with that workspace.
- If the user wants to resume the most recent conversation: `agy -c`.
- If the user wants a specific older conversation: `agy --conversation <id>`.

## Resuming a session

- Resume the most recent:
  `agy -c -i --model gemini-3.5-flash`
- Resume a specific id:
  `agy --conversation <id> -i --model gemini-3.5-flash`
- Verify a conversation exists before resuming; `agy --conversation <bogus>` fails fast.

## Starting a new session

- Only with explicit user approval.
- The agent must say, in chat, "Starting a new conversation" and confirm the workspace path before invoking `agy` without `-c` or `--conversation`.

If unsure whether to resume or start fresh, ask the user.
