## Execution surfaces — `-p` vs TUI vs resume

`agy` has three invocation shapes. The choice depends on **who is driving** (agent or user) and **whether the conversation is new or resumed**.

## 1. Headless print (`-p`)

The agent's primary surface.

```bash
agy -p "<prompt>" \
    --model "<model>" \
    --sandbox \
    --add-dir "$WORKSPACE"
```

Aliases: `-p`, `--print`, `--prompt`. Behavior: single prompt, response on stdout, exit when done. Works in non-TTY environments.

**When to use:**

- The agent is driving `agy` for the Plan or Build step.
- The task is scripted (CI, automation, batch).
- The environment is headless (no TTY).

**Capturing the response:**

```bash
OUT="$(agy -p "<prompt>" --model "<model>" --sandbox --add-dir "$WORKSPACE" 2>&1)"
echo "$OUT"
```

Or write to a file:

```bash
agy -p "<prompt>" --model "<model>" --sandbox --add-dir "$WORKSPACE" > /tmp/agy.out 2>&1
```

The response is the proposal (Plan step) or the execution report (Build step). The agent surfaces it to the human verbatim.

**Timeout:** `--print-timeout` defaults to `5m0s`. For heavy multi-file refactors, bump to `10m` or `15m`. Bump on the same line:

```bash
agy -p "<prompt>" --model "<model>" --sandbox --print-timeout 10m --add-dir "$WORKSPACE"
```

**TTY requirement:** none. This is the surface for headless servers and CI.

## 2. Interactive TUI

The user's surface. The agent does **not** drive the TUI.

```bash
agy
# or
agy -i "<initial-prompt>"
```

Aliases: `-i`, `--prompt-interactive`. Behavior: full-screen TUI (bubbletea), multi-turn, slash commands, permission prompts, artifact review panel.

**When to use:**

- The user wants to chat with `agy` directly.
- The user wants to use TUI features (slash commands, permission rules, artifact review).
- The user is doing interactive exploration.

**TTY requirement:** strict. Without a TTY, `agy` (bare) errors with `bubbletea: error opening TTY: ... open /dev/tty: no such device or address`. The agent must use `-p` in headless mode.

**What the agent should do:** instruct the user to run the TUI when they want to interact with `agy` directly. Do not try to script the TUI from the agent side.

## 3. Resume (`-c`, `--conversation <id>`)

Both shapes above can be resumed. Resume picks up an existing conversation and continues it.

```bash
# Resume the most recent conversation in this workspace
agy -c -p "<prompt>" --model "<model>" --sandbox

# Resume a specific conversation by id
agy --conversation "<id>" -p "<prompt>" --model "<model>" --sandbox
```

**When to use:**

- Step 2 of the loop (revise after a comment): `agy -c -p "..."` picks up the conversation from the Plan step.
- Step 3 of the loop (build after Approve): `agy -c -p "..."` picks up the conversation and runs the approved plan.
- The user wants to continue a previous session days later.

## Decision rule

| Situation | Surface |
|-----------|---------|
| Agent is running a plan prompt | `agy -p "..."` |
| Agent is running a build prompt after Approve | `agy -c -p "..." --dangerously-skip-permissions` |
| Agent is running a revision prompt after a comment | `agy -c -p "..."` |
| User wants to chat with `agy` directly | `agy` or `agy -i "..."` (TUI) |
| CI / scripted / headless | `agy -p "..."` |
| Resuming a previous conversation | add `-c` or `--conversation <id>` |

## Resuming across surfaces

- A conversation started in `-p` can be resumed in TUI: `agy -c -i "..."` (TUI with initial prompt) or `agy -c` (TUI, no prompt).
- A conversation started in TUI can be resumed in `-p`: `agy -c -p "..."`.
- The conversation history is shared across surfaces.

## Capturing the proposal from `-p`

The proposal is just the response text. To capture reliably:

```bash
OUT="$(agy -p "<planning-prompt>" --model "<model>" --sandbox --add-dir "$WORKSPACE" 2>&1)"
```

Then surface `$OUT` to the human verbatim in a fenced code block. Do not parse, trim, or reformat. The closing line "AWAITING APPROVAL" (or whatever the prompt asked for) is the contract.
