---
name: skill-antigravity
description: Control and operate Google Antigravity CLI (agy) via artifact-based approval. Use this skill to drive the Plan → artifact → approve → execute loop, manage sessions, select models, handle OAuth, and coordinate coding through Antigravity.
---

# skill-antigravity

## Core rule

The agent does not write code.
All planning and coding happens inside `agy` (Google Antigravity CLI).
The agent orchestrates `agy`, surfaces its markdown artifacts to the human, and waits for explicit approval before execution.

## Pre-flight

- Confirm the user has installed `agy`:
  `curl -fsSL https://antigravity.google/cli/install.sh | bash`
- Verify the binary is on PATH:
  `which agy` (expect `~/.local/bin/agy`)
- Verify authentication:
  `agy models` — succeeds = signed in; returns "Please sign in" = not authed.
- Confirm the default model is `gemini-3.5-flash` (per user decision).
- Inspect `~/.gemini/antigravity-cli/cache/projects.json` for an existing session in the current workspace. Reuse it if present; never create a new conversation without user approval.
- Mention the setup-policy choices the user made on first run: Review-driven vs Agent-driven, terminal execution policy, data-collection opt-out. If the user picked "Agent-driven" with auto-approve, warn that `agy` will run tools without asking.
- Do not proceed without the user confirming pre-flight.

## Execution surfaces

`agy` has two ways to run a task. Choose by task shape:

- **Interactive TUI** (`agy` or `agy -i "..."` or `agy --prompt-interactive "..."`): for multi-turn tasks, artifact review, permission prompts, mid-flight revisions. The agent drives a PTY, captures output, and forwards prompts to the user.
- **Headless print** (`agy --print "..." --model gemini-3.5-flash`): for one-shot tasks, single turn, no artifact review. The response is on stdout. Will still emit a markdown artifact if the prompt asks for one.

Decision rule: use the TUI whenever the task is multi-step, when revisions are expected, or when artifact review is the point. Use `--print` for one-shot code edits or short questions.

See `references/execution-surfaces.md` for the full PTY-capture pattern and the trade-offs.

## Agent mode

`agy` exposes two modes: **Fast** (default) and **Planning**. There is no `--plan` flag — the mode is set inside the TUI, or you force Planning by phrasing the first prompt as a planning request that emits a markdown artifact before any code is touched.

The agent must always force a planning pass in turn one. Concretely: the first prompt of every task is the planning prompt from `assets/operator-prompts.md`, which asks `agy` to produce a markdown artifact (typically `walkthrough.md`) without modifying any code. This gives the human a chance to approve before execution, regardless of the mode the user picked at setup time.

See `references/agent-modes.md` for the details.

## Artifact loop (the centerpiece)

`agy` Planning mode produces **artifacts** — markdown files (Task Lists, Implementation Plans, Walkthroughs, Code Diffs). The user has confirmed: "Instead of planning mode, agy will create artifact in terms of md for me to approve." The skill implements that loop as a five-step procedure:

1. **Plan** — the agent drives `agy` in TUI mode (or `--print`) and types the planning prompt from `assets/operator-prompts.md`. The prompt asks for a markdown artifact only; no code is written yet.
2. **Produce artifact** — `agy` emits the artifact. Two capture paths:
   - **(a) File output:** the artifact is written to `walkthrough.md` in the workspace (this is the path the binary references by name). The agent `cat`s it.
   - **(b) TUI output:** the agent's PTY capture buffer has the artifact text inline. The agent extracts it.
3. **Present to human** — the agent shows the artifact to the user **verbatim** in chat and closes with: *"Reply Approve to execute, or leave a comment to revise."*
4. **Incorporate comments** — if the user leaves a comment, the agent resumes the same conversation with `agy -c` (or `agy --conversation <id>`) and sends the revision prompt. `agy` re-runs planning and emits a new artifact. Loop back to step 3.
5. **Execute** — when the user replies "Approve" (or "Go", "LGTM", or any clear affirmative — when in doubt, ask), the agent resumes the same conversation and sends the approval prompt. `agy` runs the plan, emits a final `walkthrough.md`, and reports code diffs.

The agent never invents approval. It always waits for the explicit word before step 5.

Example — start a planning session:

```bash
agy -i --model gemini-3.5-flash --add-dir "$WORKSPACE"
```

Example — resume the same conversation after a revision:

```bash
agy -c -i --model gemini-3.5-flash
```

See `references/artifact-loop.md` for the full example chat reply shape, capture-path details, and the revision round-trip.

## Session management

- `agy` keeps a per-workspace history in `~/.gemini/antigravity-cli/cache/projects.json`.
- The same workspace must always use the same conversation. Reusing conversations preserves context and decisions.
- Start a new conversation only with explicit user approval.
- To resume the most recent conversation in the workspace: `agy -c`.
- To resume a specific conversation by id: `agy --conversation <id>`.
- Verify a conversation exists before resuming; `agy --conversation <bogus>` will fail.

See `references/session-management.md`.

## Model selection

- **Default:** `gemini-3.5-flash` (per user decision).
- **Per-invocation:** `agy --model <name>` or `agy -m <name>`.
- **In TUI:** type `/model` and pick from the list.
- **Verify availability:** `agy models` (auth required). Other models exposed by Antigravity include `gemini-3-pro`, `claude-sonnet-4.5`, `gpt-oss`. The agent must use a model that `agy models` actually lists — do not hard-code a list.
- If the user requests a model that is not available, surface the `agy models` output verbatim and ask the user to pick one.

See `references/model-selection.md`.

## Auth flow

- `agy` uses OAuth (no API key path). The OAuth token is stored in the system keyring.
- On first run, `agy` prints a URL like:
  ```
  Authentication required. Please visit the URL to log in:
    https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=...&...
  ```
- The URL is valid for ~30 seconds. The CLI opens a browser locally; on remote/SSH, the user copies the URL to a local browser.
- **The agent must surface the URL verbatim to the user** and not proceed without confirmation that auth completed.
- **Verify auth:** `agy models` returns the model list when authed; returns "Please sign in" when not.
- **Sign out:** open `agy`, type `/logout`.

See `references/auth-flow.md`.

## Failure handling

- **`agy` not installed** → show the install command. After install, `~/.local/bin/agy` is on PATH (may need `source ~/.bashrc`).
- **Auth URL times out (30s)** → re-invoke `agy`. The URL is regenerated. Surface it verbatim.
- **`agy models` returns "Please sign in"** → user must run `agy` interactively once to complete browser auth. Agent stops and asks the user to confirm.
- **Permission prompt mid-task** → `agy` will ask before running a tool unless the user picked auto-approve at setup. The agent driving a PTY must detect the prompt and forward it to the user. Never use `--dangerously-skip-permissions` without explicit user consent per task.
- **Sandbox blocks network** → instruct the user to retry without `--sandbox`, or change the bypass policy in `/settings`.
- **Model not available** → run `agy models`; pick from the list.
- **Conversation not found** → `agy --conversation <id>` fails. Fall back to `agy -c` (most recent) or start a new conversation only with user approval.

See `references/failure-handling.md`.

## Output format

- Show all `agy` commands and flags explicitly (no abbreviation).
- Surface OAuth URLs **verbatim** — never paraphrase or trim them.
- Surface markdown artifacts **verbatim** in fenced code blocks.
- Close every artifact presentation with: *"Reply Approve to execute, or leave a comment to revise."*
- State which conversation is in use (new vs `agy -c` vs `agy --conversation <id>`) and which model is selected.
- State the current `agy` mode (Fast vs Planning) only when the user asks; the skill forces Planning by prompt phrasing, not by mode toggle.
