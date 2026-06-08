## Description: <br>
Control and operate Google Antigravity CLI (`agy`) via artifact-based approval. Use this skill to drive the Plan → artifact → approve → execute loop, manage sessions, select models, handle OAuth, and coordinate coding through Antigravity. <br>

This skill is ready for commercial/non-commercial use. <br>

## Publisher: <br>
[AIMS](https://github.com/A-I-M-S) <br>

### License/Terms of Use: <br>
MIT <br>


## Use Case: <br>
Developers and engineers use this skill to coordinate Google Antigravity-assisted coding work through explicit OAuth confirmation, session reuse, artifact-based plan/approve rounds, and gated execution. Unlike opencode-controller, this skill bridges the absence of a `--plan` flag by forcing a markdown-artifact pass in turn one, capturing the artifact (file output or TUI buffer), and gating execution on explicit "Approve". <br>

### Deployment Geography for Use: <br>
Global <br>

## Known Risks and Mitigations: <br>
Risk: The skill coordinates an agent operating Antigravity, a closed-source Google binary, on the user's behalf. <br>
Mitigation: Document that `agy` is closed-source and not user-auditable; review every artifact before approval. <br>
Risk: Provider authentication involves OAuth login links to `accounts.google.com`. <br>
Mitigation: Surface the URL verbatim and wait for explicit confirmation; never paraphrase or trim auth URLs. <br>
Risk: `agy` runs tools (file writes, terminal commands) without per-tool permission unless the user picked "Request review" at setup. <br>
Mitigation: Confirm the user's setup policy in pre-flight; never use `--dangerously-skip-permissions` without explicit consent per task. <br>
Risk: The agent could invent approval and execute a destructive action. <br>
Mitigation: Gate step 5 of the artifact loop on the explicit word "Approve" (or clear affirmative); when in doubt, ask. <br>
Risk: OAuth token compromise via the system keyring. <br>
Mitigation: Document `/logout` to clear the token. <br>
Risk: First-run data-collection opt-in. <br>
Mitigation: Mention `/settings` and the opt-out toggle in pre-flight. <br>


## Reference(s): <br>
- [Antigravity CLI docs](https://antigravity.google/docs/cli-overview) <br>
- [Antigravity CLI repo (Apache 2.0)](https://github.com/google-antigravity/antigravity-cli) <br>
- [Command cheatsheet](references/command-cheatsheet.md) <br>
- [Artifact loop procedure](references/artifact-loop.md) <br>
- [Agent modes (Fast vs Planning)](references/agent-modes.md) <br>
- [Execution surfaces (TUI vs --print)](references/execution-surfaces.md) <br>
- [OAuth auth flow](references/auth-flow.md) <br>
- [Session management](references/session-management.md) <br>
- [Model selection procedure](references/model-selection.md) <br>
- [Standard workflow](references/workflow.md) <br>
- [Failure handling](references/failure-handling.md) <br>
- [Question handling](references/question-handling.md) <br>
- [Operator prompts](assets/operator-prompts.md) <br>


## Skill Output: <br>
**Output Type(s):** [Guidance, Markdown, Shell commands, Configuration instructions] <br>
**Output Format:** [Markdown with explicit `agy` commands, fenced artifact blocks, and concise operational instructions] <br>
**Output Parameters:** [1D] <br>
**Other Properties Related to Output:** [Includes OAuth, session, model, mode, artifact-capture, and approval-gate guidance.] <br>

## Skill Version(s): <br>
1.0.0 <br>

## Ethical Considerations: <br>
Users should evaluate whether this skill is appropriate for their environment, review every markdown artifact and the resulting code changes before approving, and apply their organization's safety, security, and compliance requirements before deployment. The skill exists to gate Antigravity's autonomous execution behind explicit human approval, not to bypass it. <br>
