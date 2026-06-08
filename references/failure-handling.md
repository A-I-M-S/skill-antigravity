## Common failures and responses

### `agy` not installed

- Show the install command:
  `curl -fsSL https://antigravity.google/cli/install.sh | bash`
- After install, the binary is at `~/.local/bin/agy`. May need `source ~/.bashrc` (or `~/.zshrc`) to refresh `PATH`.
- Verify with `which agy` and `agy --version`.

### Auth URL times out (browser flow)

- The auth URL has a short validity window. Re-run `agy` to regenerate.
- On remote/headless boxes, the URL is printed to the terminal — the user must copy it to a local browser.
- The agent does not perform OAuth on the user's behalf. It instructs the user to run `agy` interactively.

### `agy models` returns a token error

```
error getting token source: You are not logged into Antigravity.
```

- The user must run `agy` interactively once to complete browser auth.
- The agent stops, asks the user to confirm sign-in is complete, and only then resumes with `agy -c`.

### TTY errors in headless

```
CLI error: bubbletea: error opening TTY: ... open /dev/tty: no such device or address
```

- The agent must use `-p` (or `-c -p`), not the bare TUI.
- The TUI is the user's surface. The agent does not drive it.

### Permission prompt mid-task

- In `-p` mode the agent cannot forward interactive prompts to the human.
- For the Plan step (step 1), the prompt is read-only, so `agy` should not need tool execution. If it does, the planning prompt was mis-phrased.
- For the Build step (step 3), pass `--dangerously-skip-permissions` only after the proposal is approved and only with explicit user consent per task.
- For long-running work, prefer configuring permissions in the TUI via `/permissions` (TUI-only — the user does this themselves) and re-running with `--sandbox`.

### Sandbox blocks something needed

- `--sandbox` runs with terminal restrictions. If `agy` cannot do what it needs (e.g. network blocked, restricted tool), drop `--sandbox` for that invocation.
- Permissions are configured in the TUI via `/permissions`. The skill does not drive the TUI.
- Sandbox is per-invocation on the CLI; per-config in the TUI settings.

### Model not available

- Run `agy models` to list what is currently exposed.
- The agent must **not** hard-code a model list. Pick from what `agy models` returns.
- If the user asked for a model that is not listed, surface the `agy models` output verbatim and ask the user to pick one.
- Common strings (v1.0.6): `Gemini 3.1 Pro (Low)`, `Gemini 3.1 Pro (High)`, `Gemini 3.5 Flash (Low/Medium/High)`, `Gemini 3 Flash`.

### Conversation not found

- `agy --conversation <bogus>` fails. The agent must fall back to `agy -c` (most recent) or start a new conversation — but only with explicit user approval.
- If `agy -c` also fails, the cache has no record of the workspace. Start a new conversation with the user's approval.
- The cache file is `~/.gemini/antigravity-cli/cache/projects.json`; it is keyed by absolute workspace path.

### `agy` hangs or produces no output

- In `-p` mode, a hang usually means the language server is stuck or a tool call is waiting. Check the most recent log:
  `tail -f "$(ls -t ~/.gemini/antigravity-cli/log/*.log | head -1)"`
- Look for `error getting token source` (re-auth needed) or a long pause in tool-call activity.
- Bump `--print-timeout` if the task is large (default 5m; try 10m or 15m).
- If recovery is impossible, surface the hang to the user and ask whether to kill and restart the session.

### `agy` wrote files in step 1 (planning pass)

- The planning prompt should forbid file writes. If `agy` wrote a file anyway, treat it as a planning-prompt bug.
- Inspect what was written. If benign (e.g. a scratch note in `/tmp`), accept and continue.
- If destructive (e.g. it wrote code into the workspace), stop, surface to the user, do not proceed to step 3.
- Tighten the planning prompt: add "Do not call any tools. Output the proposal as text only, in a fenced code block. End with AWAITING APPROVAL."

### `--dangerously-skip-permissions` misuse

- The flag auto-approves ALL tool calls for that invocation. Misuse can produce destructive actions (rm -rf, force-push, drop tables, etc.).
- Never pass it in step 1 (planning) — the prompt is read-only.
- Pass it in step 3 (build) only after explicit **Approve** in step 2, and only with the user's consent per task.
- If the user wants auto-approve for a long session, suggest configuring permissions in the TUI via `/permissions` instead of using the flag.

### Stale settings.json

- `model` or `gcp.project` may be wrong after a project switch or model rotation. Read the file in pre-flight and surface what you find.
- Edit `settings.json` directly to change — there is no CLI flag for it.
