## Standard workflow — end to end

The five phases of every task the agent drives through `agy`. Each phase has a concrete `agy` command, an expected output, and a checkpoint with the user.

## Phase 1 — Install

One-time, on the user's host. User-driven (the agent instructs; the user runs).

```bash
curl -fsSL https://antigravity.google/cli/install.sh | bash
source ~/.bashrc
which agy          # ~/.local/bin/agy
agy --version      # 1.0.6 or later
```

## Phase 2 — Auth

The user runs `agy` interactively once to complete browser OAuth.

```bash
agy
# → CLI opens browser to Antigravity auth page
# → user signs in
# → first-run flow: pick model, project, telemetry opt-out
# → settings.json is populated
```

Verify:

```bash
agy models         # succeeds = signed in
```

**Checkpoint:** the user confirms sign-in is complete and the chosen model and project.

## Phase 3 — Pre-flight

The agent:

- Confirms `which agy` resolves and `agy --version` is in range.
- Confirms `agy models` succeeds (= authed).
- Reads `~/.gemini/antigravity-cli/settings.json` and reports the chosen `model` and `gcp.project`.
- Reads `~/.gemini/antigravity-cli/cache/projects.json` and looks for an existing conversation in the current workspace.
- Asks the user: resume an existing conversation, or start a new one?
- If `enableTelemetry` is `true`, mentions the data-collection opt-out and suggests flipping it to `false` in `settings.json`.

**Checkpoint:** the user confirms pre-flight is good and the conversation to use (new vs `-c` vs `--conversation <id>`).

## Phase 4 — Plan → revise (until Approve)

The agent runs the planning prompt (from `assets/operator-prompts.md`) via `agy -p`.

```bash
WORKSPACE="$PWD"
OUT="$(agy -p "$PLAN_PROMPT" \
    --model "Gemini 3.1 Pro (High)" \
    --sandbox \
    --add-dir "$WORKSPACE" 2>&1)"
```

`agy` returns a markdown proposal on stdout. The agent surfaces `$OUT` to the user verbatim in a fenced code block and closes with: *"Reply **Approve** to build, or leave a comment to revise."*

If the user comments, the agent resumes with `agy -c -p "$REVISE_PROMPT"` and loops until the user replies **Approve**. See `references/artifact-loop.md` for the full procedure.

## Phase 5 — Build (after explicit Approve)

The agent resumes the same conversation with the build prompt.

```bash
OUT="$(agy -c -p "$BUILD_PROMPT" \
    --model "Gemini 3.1 Pro (High)" \
    --sandbox \
    --dangerously-skip-permissions 2>&1)"
```

`agy` runs the approved plan, writes files via its built-in tools, may run shell commands, and reports results on stdout. The agent surfaces the final output to the user and reports completion.

If the user wants another iteration, restart at Phase 4 with a new planning prompt in a resumed conversation.

## End-to-end rule

> The agent never invokes `agy` for build (Phase 5) without first having surfaced a proposal (Phase 4) for the current task and received explicit **Approve**.

## Worked example (chatty version)

```text
User: refactor the cache module to split semantic vs search caches
Agent: pre-flight OK. agy 1.0.6, authed, default model Gemini 3.1 Pro (High),
       no existing conversation in this workspace. Starting a new conversation.
       Running planning pass...
       [runs agy -p with plan prompt]
       [surfaces proposal verbatim]
       Reply Approve to build, or leave a comment to revise.
User: Approve
Agent: running build...
       [runs agy -c -p with build prompt, --dangerously-skip-permissions]
       [surfaces final output]
       Done. 4 files changed, 142 insertions, 38 deletions.
```

```text
User: refactor the cache module
User: (comment after seeing the plan) use 0.92 not 0.88
Agent: revising...
       [runs agy -c -p with revise prompt]
       [surfaces revised proposal verbatim]
       Reply Approve to build, or leave a comment to revise.
User: Approve
Agent: running build...
```
