## Operator prompts

These are ready-to-paste prompts for the Plan → approve → execute loop. The agent reads them, possibly templating in the user's task, and passes them to `agy -p`.

### Plan (step 1)

Use as turn 1 of every multi-step task. Forces a markdown proposal; forbids file writes and tool calls.

```
You are planning a code change. Do NOT call any tools and do NOT modify any code.

Produce a markdown proposal with the following sections, in this order:

# plan

## Task list
- one bullet per concrete change (file, function, behavior)

## Implementation plan
- files to add or modify
- functions or classes to add or modify
- dependencies to add or remove

## Risks and trade-offs
- anything that could go wrong
- assumptions that may not hold
- things to revisit after the change is live

## Verification
- how to test the change (commands, expected output)
- any manual checks the human should do

## Open questions
- any decision you cannot make from the prompt; the human will answer

End your response with the line `AWAITING APPROVAL` on its own line. Do not include anything after that line.
```

### Revise (step 2b, after a comment)

Use when the user leaves a comment on the proposal.

```
Revise the previous plan per this comment:

<user-comment>

Re-emit the full proposal in the same format (Task list, Implementation plan, Risks, Verification, Open questions). Do NOT call any tools and do NOT modify any code. End with `AWAITING APPROVAL`.
```

### Build (step 3, after explicit Approve)

Use after the user replies Approve.

```
The previous plan is approved. Execute it exactly as proposed.

For each task in the plan:
- make the change
- run any verification command listed in the plan
- report the result

If you must deviate from the plan (because the code does not match your assumption, a dependency is missing, etc.), STOP and surface the deviation before making the change. Do not silently change scope.

When everything is done, summarize:
- files changed
- verification results
- any deviations and why
```

### Code-review (after a round of edits)

Use when the user wants a review pass against the approved plan.

```
Review the recent changes against the most recent approved plan.

Emit a markdown report with:
- pass / fail per task in the plan
- any deviations from the approved plan
- any risks you did not flag in the plan
- any leftover dead code, debug prints, or commented-out blocks

Do NOT modify any code in this step.
```

### Walkthrough (end of a build round)

Use at the end of a build round to capture the final summary in a stable format.

```
Emit a markdown walkthrough summarizing the changes from this build round:

# walkthrough

## What changed
- file-by-file summary of the diff

## How to test
- exact commands to run, with expected output

## Known follow-ups
- anything that should be cleaned up later
- anything the human should review before merging
```

### Re-auth (rare; the agent does not usually send this)

`agy` itself does not need a "re-auth prompt" — re-auth is a user action outside the conversation. The agent's job is to stop, ask the user to run `agy` interactively, and resume with `agy -c` after the user confirms.

If for some reason `agy` is in a TUI session and the user is doing the re-auth there, the user can type `/logout` and re-run `agy`. The agent does not script this.

## Templating tips

- The prompts above are wrapped in markdown code fences. Strip the outer fences when passing to `agy -p "..."`.
- For long prompts, save to a file and pass via shell substitution: `agy -p "$(cat /tmp/plan-prompt.txt)" ...`.
- Always include the closing marker (`AWAITING APPROVAL` for Plan and Revise). The agent uses it as a contract that the response is complete.
- Be explicit about "Do NOT call any tools" in Plan and Revise. `agy` will otherwise try to inspect the workspace before answering, which is fine but adds latency and may write files.
