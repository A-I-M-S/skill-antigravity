## Handling questions from `agy`

`agy` may ask two distinct kinds of questions. In `-p` mode (the agent's surface), `agy` cannot ask interactive questions — the agent must pre-empt in the prompt. In TUI mode (the user's surface), `agy` may surface interactive prompts.

### 1. `agy` asks the agent in TUI

Examples: "Should I add a test for this?", "Use SQLite or Postgres?", "Include the migration script?"

- TUI shows a prompt shape: `?` or a `[y/n]` selector.
- The user (not the agent) drives the TUI, so the user answers these directly.
- The agent does not script the TUI. If the user is in a TUI session, the agent waits for the user to report back.

### 2. The user needs to be asked (artifact comment)

When the question is about scope, design, or destructive action, the agent must NOT answer on the user's behalf. It must surface the question in chat.

- The user replies in chat.
- The agent forwards the reply as the next prompt in the same `agy` conversation via `agy -c -p "..."`.
- This becomes a revise round in the Plan → approve → execute loop.

### 3. Pre-empting questions in `-p` mode

`agy` cannot ask interactive questions in `-p` mode. The agent must put all required decisions into the planning prompt up front. Example:

> Before writing the plan, decide and document:
> - Test framework (assume pytest unless I say otherwise).
> - Whether to add a migration script (assume yes unless I say otherwise).
> - The default similarity threshold for the semantic cache (use 0.88, document as a tunable).
> If you cannot decide, list the open question in the proposal and end with AWAITING APPROVAL.

This keeps `-p` mode flow going without mid-stream pauses.

## Never invent a "yes" on destructive actions

- Deleting a file, force-pushing, running a deploy, dropping a database, or any other irreversible action requires explicit user confirmation.
- The agent must NOT type `y` or `yes` into a destructive prompt without the user having said so in chat.
- When in doubt, escalate.

## Mid-execution question from `agy` (Build step)

If `agy` is mid-build (post-Approve) and surfaces a question (in `-p` mode, this would be in the final report):

- The Build step is one shot in `-p` mode. The agent cannot pause and resume mid-stream.
- If the question is in the final report, surface it to the user.
- If the question requires a code change, run a new Plan round (step 1 again) with the question in the prompt.
- Do NOT modify the approved plan in step 3. The plan is approved as-is; deviations should be reported, not silently made.

## What the agent does when the user asks for the TUI

If the user wants to interact with `agy` directly (TUI), the agent steps back:

- The user runs `agy` (or `agy -i "..."`) themselves.
- The agent waits for the user to report back.
- When the user reports back, the agent picks up the conversation via `agy -c -p "..."` and continues the loop in `-p` mode.

The agent does not try to script the TUI from outside.
