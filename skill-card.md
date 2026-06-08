## Description:
Control and operate Google Antigravity CLI (`agy`) via an explicit Plan → approve → execute loop. Use this skill to drive `agy` headless (`-p`), capture its markdown proposals, surface them to the human for approval, and only then run the build. The skill is validated against `agy` v1.0.6 and reflects the real CLI surface (no invented subcommands).

## Publisher:
[AIMS](https://github.com/A-I-M-S)

### License/Terms of Use:
MIT

## Use Case:
Developers and engineers use this skill to coordinate Google Antigravity-assisted coding work through explicit OAuth confirmation, session reuse, plan/approve rounds, and gated execution. The skill bridges `agy`'s lack of a `--plan` flag by forcing a planning pass in the first prompt, capturing `agy`'s markdown proposal on stdout, and gating the build on an explicit **Approve**. Unlike `opencode-controller`, this skill drives the headless `-p` surface (not a TUI) and is shaped around `agy`'s real flag/command surface.

## Deployment Geography for Use:
Global

## Known Risks and Mitigations:
**Risk:** The skill coordinates an agent operating Antigravity, a closed-source Google binary, on the user's behalf.
**Mitigation:** Document that `agy` is closed-source and not user-auditable; review every proposal before approval.

**Risk:** Provider authentication involves OAuth login to a Google account via the Antigravity app.
**Mitigation:** The agent does not perform OAuth on the user's behalf. It instructs the user to run `agy` interactively to complete auth, then verifies with `agy models`.

**Risk:** `agy` runs tools (file writes, terminal commands) without per-tool permission when `--dangerously-skip-permissions` is set, or when the user picked "Request review" mode that still allows tool use.
**Mitigation:** Pass `--dangerously-skip-permissions` only in the build step (step 3), only after the proposal is approved in step 2, and only with explicit user consent per task.

**Risk:** The agent could invent approval and execute a destructive action.
**Mitigation:** Gate step 3 on the explicit word **Approve** (or clear affirmative). When in doubt, ask.

**Risk:** OAuth token compromise via the local token file.
**Mitigation:** Document `/logout` (TUI) to clear the token from `~/.gemini/antigravity-cli/antigravity-oauth-token`.

**Risk:** First-run data-collection opt-in.
**Mitigation:** Mention the `enableTelemetry` field in `settings.json` in pre-flight.

## Reference(s):
- [Antigravity CLI docs](https://antigravity.google/docs/cli-overview)
- [Antigravity CLI repo (Apache 2.0)](https://github.com/google-antigravity/antigravity-cli)
- [Command cheatsheet](references/command-cheatsheet.md)
- [Artifact loop procedure](references/artifact-loop.md)
- [Execution surfaces (-p vs TUI vs resume)](references/execution-surfaces.md)
- [OAuth auth flow](references/auth-flow.md)
- [Session management](references/session-management.md)
- [Model selection procedure](references/model-selection.md)
- [Standard workflow](references/workflow.md)
- [Failure handling](references/failure-handling.md)
- [Question handling](references/question-handling.md)
- [Operator prompts](assets/operator-prompts.md)

## Skill Output:
**Output Type(s):** [Guidance, Markdown, Shell commands, Configuration instructions]
**Output Format:** [Markdown with explicit `agy` commands, fenced artifact blocks, and concise operational instructions]
**Output Parameters:** [1D]
**Other Properties Related to Output:** [Includes OAuth, session, model, artifact-capture, and approval-gate guidance.]

## Skill Version(s):
1.0.1

## Ethical Considerations:
Users should evaluate whether this skill is appropriate for their environment, review every markdown proposal and the resulting code changes before approving, and apply their organization's safety, security, and compliance requirements before deployment. The skill exists to gate Antigravity's autonomous execution behind explicit human approval, not to bypass it.
