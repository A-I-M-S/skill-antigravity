## Artifact loop — the skill's centerpiece

`agy` Planning mode produces **artifacts**: markdown files (Task Lists, Implementation Plans, Walkthroughs, Code Diffs, Screenshots, Browser Recordings). The user has confirmed the workflow: "Instead of planning mode, agy will create artifact in terms of md for me to approve." This document defines the five-step loop the agent runs.

## Step 1 — Plan

The agent drives `agy` in interactive mode and types (or pipes) the planning prompt from `assets/operator-prompts.md`. The first prompt of every task is a planning prompt — no code is written until the user approves the artifact.

```bash
agy -i --model gemini-3.5-flash --add-dir "$WORKSPACE"
```

The planning prompt asks `agy` to produce a markdown artifact titled `walkthrough.md` containing: a task list, an implementation plan (files, functions, dependencies), risks, and a verification approach. It explicitly forbids code modification.

## Step 2 — Produce artifact

`agy` runs in Planning mode and emits a markdown artifact. There are two capture paths:

- **(a) File output** — the artifact is written to `walkthrough.md` in the workspace (this is the path the binary references by name). The agent reads it with `cat walkthrough.md` or by opening the file in the editor.
- **(b) TUI output** — the agent's PTY capture buffer has the artifact text inline. The agent extracts the markdown block from the scrollback.

Path (a) is the canonical capture. Path (b) is the fallback when the binary doesn't write a file (e.g. `--print` mode, or when the prompt is mis-phrased and `agy` answers inline).

## Step 3 — Present to human

The agent shows the artifact to the user **verbatim** in chat, in a fenced markdown code block, and closes with the standard ask:

> Here is `agy`'s plan for the change. Please review and reply **Approve** to execute, or leave a comment to revise.
>
> ```markdown
> # walkthrough
>
> ## Task list
> - [ ] Add `cache.py` with `SemanticCache` + `SearchCache`
> - [ ] Add 10 new `Settings` fields to `config.py`
>
> ## Implementation plan
> ...
>
> ## Risks
> - 0.88 similarity threshold is a guess; tune later
> ```

The closing line is required on every artifact presentation. It is the contract that gates step 5.

## Step 4 — Incorporate comments

If the user leaves a comment instead of "Approve" (e.g. "Use 0.92 instead of 0.88"), the agent resumes the same conversation and forwards the comment as a revision prompt.

```bash
agy -c -i --model gemini-3.5-flash
```

Then types (or pipes) the revision prompt:

> "Revise `walkthrough.md` per this comment: change `SEMANTIC_CACHE_SIMILARITY_THRESHOLD` from 0.88 to 0.92. Re-emit the artifact. Do NOT modify any code yet."

`agy` revises the artifact. Loop back to step 3. The agent may iterate the revise-present loop as many times as the user wants.

## Step 5 — Execute

When the user replies "Approve" (or "Go", "LGTM", "Ship it", or any clear affirmative — when in doubt, ask), the agent resumes the same conversation and sends the approval prompt.

```bash
agy -c -i --model gemini-3.5-flash
```

Then types (or pipes) the approval prompt:

> "All items in `walkthrough.md` are approved. Proceed with the implementation plan. Report any deviations as you go."

The agent lets `agy` run. `agy` may emit intermediate status, may ask permission to run tools (the agent driving a PTY must detect these and surface them to the user — see `references/question-handling.md`), and finishes by emitting a final `walkthrough.md` plus code diffs.

## The approval gate — explicit words

The agent never invents approval. It waits for one of these from the human, verbatim or paraphrased:

- "Approve"
- "Go"
- "LGTM"
- "Ship it"
- "Yes, proceed"

When in doubt — if the user says "looks good but...", "almost", or sends only a partial revision — the agent must re-confirm. The cost of one extra confirmation is much lower than executing a destructive action without consent.

## Why no `--plan` flag

`agy` has no `--plan` CLI flag. The skill bridges this by:

1. Always using the planning prompt as turn 1 (forces a planning pass regardless of mode).
2. Capturing the artifact from `walkthrough.md` (or the TUI buffer) and presenting it verbatim.
3. Gating execution on the explicit word "Approve".

The agent must never invoke `agy` for execution without first having read and surfaced a `walkthrough.md` (or equivalent artifact) for the current task.
