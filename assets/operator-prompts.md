### Planning prompt

Use as turn 1 of every multi-step task. Forces a markdown artifact; forbids code modification.

```
Produce a markdown artifact titled `walkthrough.md` in the current workspace containing:
- a task list (one bullet per concrete change)
- an implementation plan (files, functions, dependencies)
- risks and trade-offs
- a verification approach (how to test the change)

Do NOT modify any code yet. Wait for my approval before executing.
```

### Approval prompt

Use after the user replies "Approve" to a planning artifact.

```
All items in `walkthrough.md` are approved. Proceed with the implementation plan. Report any deviations as you go.
```

### Revision prompt

Use when the user leaves a comment on the artifact.

```
Revise `walkthrough.md` per this comment: <comment>. Re-emit the artifact. Do NOT modify any code yet.
```

### Code-review prompt

Use after a round of edits to check the implementation against the approved plan.

```
Review the recent changes against `walkthrough.md`. Emit a markdown report with:
- pass / fail per task
- any deviations from the approved plan
- any risks you did not flag in the plan
```

### Walkthrough prompt

Use at the end of an execution round to capture the final summary.

```
When the implementation is complete, emit `walkthrough.md` summarizing the changes and how to test them.
```

### Re-auth prompt

Use when `agy models` returns "Please sign in" mid-task.

```
The session is unauthenticated. Please surface the OAuth URL so I can complete sign-in before continuing. Do not run any tools.
```
