## Common failures and responses

### `agy` not installed
- Show the install command:
  `curl -fsSL https://antigravity.google/cli/install.sh | bash`
- After install, the binary is at `~/.local/bin/agy`. May need `source ~/.bashrc` (or `~/.zshrc`) to refresh `PATH`.
- Verify with `which agy` and `agy --version`.

### Auth URL times out (30 seconds)
- The URL is regenerated on each `agy` invocation. Re-run `agy` and surface the new URL verbatim.
- Do not paraphrase or trim the URL — see `references/auth-flow.md`.

### `agy models` returns "Please sign in"
- The user must run `agy` interactively once to complete browser auth.
- The agent stops, asks the user to confirm sign-in is complete, and only then resumes with `agy -c`.

### Permission prompt mid-task
- `agy` will ask before running a tool unless the user picked auto-approve at setup.
- The agent driving a PTY must detect the prompt and forward it to the user — see `references/question-handling.md`.
- Never use `--dangerously-skip-permissions` without explicit user consent per task. The flag auto-approves ALL tool calls; misuse can produce destructive actions.

### Sandbox blocks network
- If `--sandbox` is set and `agy` cannot reach the model API, instruct the user to retry without `--sandbox`, or open `/settings` and change the sandbox bypass policy.
- Sandbox is per-invocation on the CLI; per-config in the TUI settings panel.

### Model not available
- Run `agy models` to list what is currently exposed.
- The agent must NOT hard-code a model list — pick from what `agy models` actually returns.
- If the user asked for a model that is not listed, surface the `agy models` output verbatim and ask the user to pick one.

### Conversation not found
- `agy --conversation <bogus>` fails. The agent must fall back to `agy -c` (most recent) or start a new conversation — but only with explicit user approval.
- If `agy -c` also fails, run `agy` with no flags to see the conversation selector (`/resume` does the same in the TUI).

### `agy` hangs or produces no output
- Check the PTY capture buffer for a permission prompt or a tool-approval dialog.
- If `agy` is stuck in a tool loop, type `/diff` to see pending changes, then `/permissions` to review the policy.
- If recovery is impossible, the agent must surface the hang to the user and ask whether to kill and restart the session.
