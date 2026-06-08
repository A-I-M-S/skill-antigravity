---
name: skill-antigravity
description: Drive Google Antigravity CLI (agy) via operator prompts and an explicit Plan → approve → execute loop. Use this skill to run agy headless (-p), capture its markdown proposals, surface them to the human for approval, and only then run the build.
---

# skill-antigravity

## Core rule

The agent does not write code.

The agent drives `agy` (Google Antigravity CLI) through operator prompts. For every non-trivial task, the agent runs three `agy -p` invocations:

1. **Plan** — operator prompt asks `agy` for a markdown proposal. No files are written.
2. **Approve** — human reviews the proposal and replies **Approve** (or leaves a comment for a revision round).
3. **Build** — operator prompt tells `agy` the proposal is approved and to execute it.

The agent never invents approval. Step 3 never runs without an explicit **Approve** in step 2.

## Pre-flight

Before driving `agy`, the agent confirms:

- `agy` is on PATH:
  `which agy` (expect `~/.local/bin/agy`)
- Version is in range:
  `agy --version` (this skill is validated against v1.0.6; older versions may have a different surface)
- The user is authenticated:
  `agy models` succeeds when authed; returns a token error when not.
- The user's chosen model and project:
  `cat ~/.gemini/antigravity-cli/settings.json` → `model`, `gcp.project`
- The current workspace's existing session (if any):
  `cat ~/.gemini/antigravity-cli/cache/projects.json`
- The setup policy: did the user opt out of telemetry? Is `enableTelemetry` set to `false` in `settings.json`? If not, mention it.

The agent mentions these facts to the user and asks for confirmation before driving `agy`.

**Install (one-time, user-driven):**

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
source ~/.bashrc
which agy   # ~/.local/bin/agy
```

## Execution surfaces

`agy` has three invocation shapes. Choose by who is driving and what the task looks like.

- **Headless print** — `agy -p "..."` / `agy --print "..."` / `agy --prompt "..."`
  Single prompt, response on stdout, exits when done. Works in non-TTY environments. This is the agent's primary surface.
- **Interactive TUI** — `agy` / `agy -i "..."` / `agy --prompt-interactive "..."`
  Full-screen TUI (bubbletea). Needs a TTY; errors with `could not open /dev/tty` in headless. The user runs this directly when they want to chat with `agy` themselves. The agent does **not** drive the TUI.
- **Resume a conversation** — `agy -c` / `agy --continue` / `agy --conversation <id>`
  Continues an existing conversation. Compatible with both `-p` and TUI. Use `-c` for the most recent in the current workspace, `--conversation <id>` for a specific one.

Decision rule: use `-p` for everything the agent drives. Use TUI when the user wants to interact with `agy` directly. Use `-c` / `--conversation` to resume for revision rounds and builds.

See `references/execution-surfaces.md`.

## Plan → approve → execute (the centerpiece)

The skill runs every non-trivial task through three `agy -p` invocations. The prompts come from `assets/operator-prompts.md`.

### 1. Plan

```bash
agy -p "$(cat assets/operator-prompts.md::plan)" \
    --model "<exact-string-from-agy-models>" \
    --sandbox \
    --add-dir "$WORKSPACE"
```

`agy` returns a markdown proposal on stdout. The agent surfaces it to the human **verbatim** in a fenced code block, closing with:

> Reply **Approve** to build, or leave a comment to revise.

### 2. Approve (or revise)

- If the human replies **Approve** (or "Go", "LGTM", "Ship it", or any clear affirmative), go to step 3.
- If the human leaves a comment, resume the conversation and send a revision prompt:

```bash
agy -p "$(cat assets/operator-prompts.md::revise)" -c \
    --model "<model>" \
    --sandbox
```

Loop back to step 1 with the new proposal.

### 3. Build

```bash
agy -p "$(cat assets/operator-prompts.md::build)" -c \
    --model "<model>" \
    --sandbox \
    --dangerously-skip-permissions
```

`agy` runs the plan, may write files via its built-in tools, may run shell commands, and reports results on stdout. The agent surfaces the final output to the human.

The agent never invents approval. `--dangerously-skip-permissions` is used in step 3 only, and only after the human has explicitly approved the proposal in step 2.

See `references/artifact-loop.md` for the full example, the revision round-trip, and the approval-gate rules.

## Session management

- `agy` keeps a per-workspace history in `~/.gemini/antigravity-cli/cache/projects.json`.
- Reuse conversations: `-c` for the most recent in the current workspace, `--conversation <id>` for a specific one.
- Verify a conversation exists before resuming; `agy --conversation <bogus>` fails fast.
- Start a new conversation only with explicit user approval.
- The same workspace must always use the same conversation. Reusing conversations preserves context and decisions.

See `references/session-management.md`.

## Model selection

- **Default:** the value in `~/.gemini/antigravity-cli/settings.json` → `model`. As of v1.0.6, common values are `Gemini 3.1 Pro (High)`, `Gemini 3.1 Pro (Low)`, `Gemini 3.5 Flash (Low)`, `Gemini 3.5 Flash (Medium)`, `Gemini 3.5 Flash (High)`, `Gemini 3 Flash`. Exact strings include the effort suffix in parentheses.
- **Verify the current default:** `grep '"model"' ~/.gemini/antigravity-cli/settings.json`.
- **Per-invocation override:** `agy --model "<name>"` — use the exact string from `agy models`.
- **List available models:** `agy models` (auth required).
- The agent must **not** hard-code a model list. Run `agy models` and use one it returns. If the user asked for a model that is not listed, surface the `agy models` output verbatim and ask the user to pick one.

See `references/model-selection.md`.

## Auth flow

- `agy` uses OAuth (no API-key path). The token is stored at `~/.gemini/antigravity-cli/antigravity-oauth-token` (mode 0600).
- On first run (or after `/logout`), `agy` requires a browser OAuth flow. The CLI opens a local browser to the Antigravity auth page. On remote/headless boxes, the user must copy the URL to a local browser.
- **The agent does not perform OAuth on the user's behalf.** It instructs the user to run `agy` interactively to complete auth, then verifies with `agy models`.
- **Verify auth:** `agy models` succeeds when authed; returns a token-source error when not.
- **Sign out:** in the TUI, type `/logout` (TUI-only — there is no CLI flag for sign-out).
- **Project id** is set in `settings.json` → `gcp.project` (not a CLI flag).
- **First-run settings** are stored in `~/.gemini/antigravity-cli/settings.json`:
  ```json
  {
    "enableTelemetry": false,
    "gcp": {"project": "<project-id>", "location": "global"},
    "model": "Gemini 3.1 Pro (High)"
  }
  ```
  Edit this file directly to change project, model, or telemetry.

See `references/auth-flow.md`.

## Failure handling

- **`agy` not installed** → show the install command. After install, `~/.local/bin/agy` is on PATH; may need `source ~/.bashrc`.
- **`agy models` returns a token error** → user must run `agy` interactively to complete browser auth. Agent stops, asks the user to confirm, then resumes.
- **Permission prompt mid-task** → in `-p` mode the agent cannot forward interactive prompts. Pass `--dangerously-skip-permissions` only after the proposal is approved (step 3) and only with explicit user consent per task.
- **Sandbox blocks something needed** → drop `--sandbox` for that invocation, or configure permissions in the TUI via `/permissions` (TUI-only).
- **Model not available** → run `agy models`; pick from the list.
- **Conversation not found** → `agy --conversation <bogus>` fails. Fall back to `-c`, or start a new conversation only with user approval.
- **TTY errors in headless** → the agent must use `-p` (or `-c -p`), not the bare TUI.

See `references/failure-handling.md`.

## Output format

- Show all `agy` commands and flags explicitly.
- Surface `agy`'s response **verbatim** in fenced code blocks.
- Close every proposal presentation with: *"Reply **Approve** to build, or leave a comment to revise."*
- State which conversation is in use (new vs `-c` vs `--conversation <id>`) and which model is selected.
- Quote OAuth URLs verbatim when surfacing them — never paraphrase or trim.
