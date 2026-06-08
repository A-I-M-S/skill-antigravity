## The Plan → approve → execute loop

This is the skill's centerpiece. Every non-trivial task runs through three `agy -p` invocations. The prompts come from `assets/operator-prompts.md`.

The "artifact" is `agy`'s response to the planning prompt: a markdown proposal. It is not a special binary output — it is just the text `agy` writes to stdout in `-p` mode. The skill captures it, surfaces it to the human, and gates the build on explicit approval.

## Step 1 — Plan

The agent runs `agy -p` with the planning prompt. The prompt asks for a markdown proposal and forbids any file writes. `agy` returns the proposal on stdout.

```bash
PROMPT="$(cat assets/operator-prompts.md | awk '/^### Plan$/,/^### /{ if (!/^### / || /^### Plan$/) print }')"
OUT="$(agy -p "$PROMPT" --model "Gemini 3.1 Pro (High)" --sandbox --add-dir "$WORKSPACE" 2>&1)"
```

(Or just paste the prompt inline; the `awk` example is for templating.)

The planning prompt must end with a clear marker — typically `AWAITING APPROVAL` — so the response has a recognizable end. The agent verifies the marker is present; if not, surface what `agy` actually said and ask the user how to proceed.

**Capture shape:**

```text
[markdown proposal]
...
AWAITING APPROVAL
```

## Step 2 — Surface to the human

The agent shows the proposal to the user **verbatim** in chat, in a fenced code block, and closes with the standard ask:

> Here is `agy`'s plan for the change. Please review and reply **Approve** to build, or leave a comment to revise.
>
> ```markdown
> # plan
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
>
> AWAITING APPROVAL
> ```

The closing line is required on every proposal presentation. It is the contract that gates step 3.

## Step 2b — Revise (if the human comments)

If the user leaves a comment instead of "Approve" (e.g. "Use 0.92 instead of 0.88"), the agent resumes the same conversation with a revision prompt:

```bash
agy -c -p "Revise the previous plan per this comment: use 0.92 instead of 0.88. Re-emit the full proposal in the same format. Do NOT modify any code yet. End with AWAITING APPROVAL." \
    --model "Gemini 3.1 Pro (High)" --sandbox
```

`agy` re-emits the proposal. Loop back to step 2 with the new proposal. The agent may iterate the revise–present loop as many times as the user wants.

## Step 3 — Build (after explicit Approve)

When the user replies **Approve**, the agent resumes the same conversation and sends the build prompt.

```bash
agy -c -p "The plan is approved. Execute it as proposed. Report any deviations from the plan as you go. When done, summarize the changes." \
    --model "Gemini 3.1 Pro (High)" \
    --sandbox \
    --dangerously-skip-permissions
```

`agy` runs the plan, may write files, may run shell commands, and reports results on stdout. The agent surfaces the final output to the human.

**`--dangerously-skip-permissions` is used here only**, after approval. Do not use it in step 1 or step 2b — those are read-only proposals and don't need tool execution; if `agy` tries to run a tool in step 1, that is a sign the planning prompt was mis-phrased.

## The approval gate — explicit words

The agent never invents approval. It waits for one of these from the human, verbatim or paraphrased:

- **Approve**
- Go
- LGTM
- Ship it
- Yes, proceed

When in doubt — if the user says "looks good but...", "almost", or sends only a partial revision — the agent must re-confirm. The cost of one extra confirmation is much lower than executing a destructive action without consent.

## Why no `--plan` flag

`agy` has no `--plan` CLI flag and no mode toggle. The skill bridges this by:

1. Always sending a planning prompt as step 1 (forces a planning pass).
2. Capturing the proposal from `-p` stdout and presenting it verbatim.
3. Gating step 3 on the explicit word **Approve**.

The agent must never invoke `agy` for build (step 3) without first having read and surfaced a proposal (step 1) for the current task and received explicit **Approve**.

## The "no file writes" rule for step 1

The planning prompt explicitly forbids file writes. If `agy` writes a file in step 1, the agent should:

1. Note the deviation.
2. Decide whether the file is benign (e.g. a scratch note in `/tmp`) or destructive (e.g. it wrote code into the workspace).
3. If destructive, surface to the user, do not proceed to step 3, and revise the planning prompt to be stricter (e.g. add "Do not call any tools. Output the proposal as text only.").

In practice, the marker "Do not modify any code yet" + "Output the proposal as text in a fenced code block, do not call any tools" is enough to keep `agy` honest in step 1. See `assets/operator-prompts.md`.
