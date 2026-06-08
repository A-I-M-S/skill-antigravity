## Execution surfaces — TUI vs `--print`

`agy` exposes two ways to run a task. The choice depends on the task shape and on whether artifact review is expected.

## Interactive TUI

- Invoked by: `agy`, `agy -i "..."`, `agy --prompt-interactive "..."`.
- For: long sessions, multi-turn work, artifact review, mid-flight revisions, permission prompts.
- Behavior: opens a full-screen TUI (bubbletea, same family as opencode). The agent drives a PTY and captures output. TUI slash commands (`/model`, `/resume`, `/review`, `/permissions`, etc.) are available.

PTY capture pattern for the agent:

```bash
script -q -c "agy -i --model gemini-3.5-flash --add-dir \"$WORKSPACE\"" /tmp/agy.log
# or, in a real openclaw agent, allocate a PTY via the standard library
# and pipe the planning prompt to stdin after `agy` boots.
```

The agent reads `/tmp/agy.log` (or the PTY buffer) to extract the artifact, then forwards prompts by writing to the PTY.

## Headless `--print`

- Invoked by: `agy --print "..."`, `agy -p "..."`, `agy --prompt "..."`.
- For: one-shot tasks, single turn, scripted flows, CI.
- Behavior: response is on stdout. Exit code reflects success/failure. Default timeout 5m0s; override with `--print-timeout <duration>`.
- Will still emit a markdown artifact if the prompt asks for one — capture is the stdout text.

```bash
agy --print "Produce walkthrough.md for the cache refactor. Do NOT modify any code." \
    --model gemini-3.5-flash \
    --add-dir "$WORKSPACE" \
    --print-timeout 5m
```

## Decision rule

| Task shape                                | Surface   |
|-------------------------------------------|-----------|
| One-shot edit, single question            | `--print` |
| Multi-step change with artifact review    | TUI       |
| Revision round (incorporate user comment) | TUI       |
| Permission prompt expected mid-task       | TUI       |
| CI / scripted automation                  | `--print` |

When in doubt, use the TUI. The TUI supports everything `--print` does, plus artifact review and permission handling. The only cost is the PTY setup.

## Capturing artifacts from each surface

- **TUI:** artifact is in `walkthrough.md` (file output) or in the PTY scrollback. Read the file first; fall back to the scrollback.
- **`--print`:** artifact is on stdout. Capture with `agy --print "..." > /tmp/agy.out 2>&1` and parse the markdown block.

## Resuming across surfaces

- A conversation started in TUI can be resumed with `agy -c` (TUI) or `agy -c --print "..."` (`--print`).
- A conversation started in `--print` can be resumed with `agy -c -i` (TUI) for follow-up artifact review.
- The conversation history is shared.
