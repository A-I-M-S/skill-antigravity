## Agent modes in `agy`

`agy` has two modes: **Fast** (default) and **Planning**. Unlike opencode, there is no `--plan` flag and no clean CLI toggle. The mode is set inside the TUI or forced by prompt phrasing.

## Fast mode

- Default. Skips planning, answers and edits directly.
- Suitable for: trivial edits, one-line fixes, "what does this function do" questions.
- The skill does NOT use Fast for multi-step tasks — it would skip the artifact pass.

## Planning mode

- Thinks before acting. Produces a markdown artifact (Task List, Implementation Plan, Walkthrough, Code Diff) before any code is written.
- Suitable for: any non-trivial change, refactor, new feature, multi-file edit.
- The skill forces Planning for every multi-step task by always sending a planning prompt as turn 1.

## Forcing Planning without a flag

There are two ways:

- **(a) TUI mode-switch.** Open `agy`, press the mode key to switch to Planning, then type the planning prompt.
- **(b) Prompt phrasing.** Send the planning prompt from `assets/operator-prompts.md` as turn 1. The prompt explicitly asks for a markdown artifact and forbids code modification. `agy` honors the request and produces the artifact, regardless of the active mode.

The skill standardizes on (b) because it works in both TUI and `--print` modes and does not require the agent to send a mode-key escape sequence through a PTY.

## The skill's invariant

> The first prompt of every task is a planning prompt. The agent never invokes `agy` for execution without first having received an approved markdown artifact for the current task.

This invariant holds whether `agy` is in Fast or Planning mode, whether the user is on the TUI or `--print`, and whether the conversation is new or resumed.

## When the user explicitly wants Fast mode

If the user says "just do it, no plan", the agent still sends a one-line artifact prompt (a single-task plan) and waits for the brief "Approve" before executing. The artifact loop is not optional — it is the safety gate. A one-bullet plan takes seconds to approve and prevents accidental destructive actions.
