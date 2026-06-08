## Standard workflow — end to end

The five phases of every task the agent drives through `agy`. Each phase has a concrete `agy` command, an expected output, and a checkpoint with the user.

## Phase 1 — Install

One-time, on the user's host. Agent-driven or user-driven.

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
source ~/.bashrc
which agy   # expect ~/.local/bin/agy
agy --version
```

## Phase 2 — Auth

The user runs `agy` interactively once to complete browser OAuth.

```bash
agy
# → prints URL, opens browser, user signs in
# → first-run setup flow: pick Secure / Review-driven / Agent-driven / Custom
```

Verify:

```bash
agy models   # returns model list = signed in
```

**Checkpoint:** the user confirms sign-in is complete and the chosen setup policy.

## Phase 3 — Pre-flight

The agent:

- Confirms `which agy` resolves.
- Confirms `agy models` succeeds (= authed).
- Confirms the default model is `gemini-3.5-flash` (or asks the user to confirm a different one).
- Reads `~/.gemini/antigravity-cli/cache/projects.json` and looks for an existing conversation in the current workspace.
- Mentions `/settings` and the data-collection opt-out if the user has not seen it.
- Confirms the user's setup policy (Review-driven vs Agent-driven). If Agent-driven with auto-approve, warn that `agy` will run tools without asking.

**Checkpoint:** the user confirms pre-flight is good and the conversation to use (new vs `agy -c`).

## Phase 4 — Plan → artifact → approve

The agent drives a planning pass. First prompt is always a planning prompt.

```bash
agy -i --model gemini-3.5-flash --add-dir "$WORKSPACE"
# types the planning prompt from assets/operator-prompts.md
```

`agy` produces a markdown artifact (Task List, Implementation Plan, Walkthrough, Code Diff). The agent reads `walkthrough.md` (or extracts the TUI buffer).

The agent presents the artifact to the user verbatim and closes with: *"Reply Approve to execute, or leave a comment to revise."*

If the user comments, the agent resumes with `agy -c` and sends the revision prompt. Loop until the user replies "Approve".

## Phase 5 — Execute

The agent resumes the same conversation and sends the approval prompt.

```bash
agy -c -i --model gemini-3.5-flash
# types the approval prompt from assets/operator-prompts.md
```

`agy` runs the plan, asks permission for tools as needed (the agent forwards prompts to the user), and finishes with a final `walkthrough.md` plus code diffs.

The agent surfaces the final artifact to the user and reports completion. If the user wants another iteration, restart at Phase 4 with a new planning prompt in a resumed conversation.

## End-to-end rule

> The agent never invokes `agy` for execution without first having read and surfaced a `walkthrough.md` (or equivalent artifact) for the current task and received explicit "Approve".
