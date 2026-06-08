## Handling questions from `agy`

`agy` may ask two distinct kinds of questions:

### 1. `agy` asks the agent (in TUI)

Examples: "Should I add a test for this?", "Use SQLite or Postgres?", "Include the migration script?"

- The agent driving the PTY must detect the question (the prompt shape is `?` or a `[y/n]` selector in the TUI).
- The agent decides if it can answer itself (no user input needed, e.g. project-convention question) or if it must escalate to the user.
- For self-answerable questions, the agent types the answer into the PTY and continues.
- For questions that need user judgment, the agent stops, surfaces the question verbatim in chat, and waits for the user to reply.

### 2. The user needs to be asked (artifact comment)

When the question is about scope, design, or destructive action, the agent must NOT answer in TUI. It must surface the question as a comment on the current artifact (or, if there is no current artifact, as a chat question to the user).

- The user replies in chat.
- The agent forwards the reply as the next prompt in the same `agy` conversation (`agy -c`).

## Never invent a "yes" on destructive actions

- Deleting a file, force-pushing, running a deploy, dropping a database, or any other irreversible action requires explicit user confirmation.
- The agent must NOT type `y` or `yes` into a destructive prompt without the user having said so in chat.
- When in doubt, escalate.

## Mid-execution question from `agy`

If `agy` is mid-execution (post-approval) and asks a question, the agent should:

- Pause. Surface the question to the user verbatim.
- Wait for the user's answer.
- Forward the answer to `agy` as the next prompt in the same conversation.
- Do NOT switch back to a planning pass unless the user's answer changes the plan materially.
