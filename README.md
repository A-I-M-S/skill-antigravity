# skill-antigravity

OpenClaw skill for controlling **Google Antigravity CLI** (`agy`) — Google's agentic development platform (Nov 2025, VS Code fork + agent-first Mission Control).

This is an **operator-prompt skill**: pure markdown, no Python, no dependencies. It teaches an openclaw agent how to drive `agy` from a terminal through a structured **Plan → artifact → approve → execute** loop, then how to surface the result back to the human.

**This is NOT a code library.** There are no Python modules to import, no API to call. The skill is a markdown contract the consuming agent follows.

## What this skill does

- Wraps the `agy` CLI (TUI and `--print`) for headless and non-interactive orchestration.
- Drives the **Plan → Artifact → Approve → Execute** loop:
  1. **Plan** — the agent sends the task to `agy` with a planning prompt that asks for a markdown artifact (`walkthrough.md`), no code yet.
  2. **Artifact** — `agy` writes the artifact to disk (or emits it in the TUI). The agent captures it.
  3. **Approve** — the agent surfaces the artifact to the human **verbatim** and waits for the explicit word "Approve" (or a clear affirmative).
  4. **Execute** — once approved, the agent resumes the same `agy` conversation with the approval prompt; `agy` runs the plan and emits the final `walkthrough.md` plus code diffs.
- Manages session reuse (`agy -c`, `agy --conversation <id>`), model selection (default `gemini-3.5-flash`), and OAuth handoff.
- Closes the gap left by `agy`'s lack of a `--plan` flag: planning is forced by prompt phrasing, not by mode toggle.

## Install

The skill is a directory of markdown files consumed by an openclaw agent. To use it:

```bash
# Clone the skill
git clone https://github.com/A-I-M-S/skill-antigravity.git
cd skill-antigravity

# The skill itself needs no install. The CLI it drives does:
curl -fsSL https://antigravity.google/cli/install.sh | bash
source ~/.bashrc
```

## Quick start

```bash
# 1. Verify the binary
which agy          # expect ~/.local/bin/agy

# 2. Authenticate (one-time, browser OAuth)
agy
# → prints URL, opens browser, user signs in

# 3. Verify auth
agy models         # returns the model list = signed in

# 4. Read the procedure
# The agent reads SKILL.md and follows the five-step loop.
# See references/workflow.md for the end-to-end walkthrough.
```

## Layout

| File | Purpose |
|------|---------|
| `SKILL.md` | Main procedure the agent follows. |
| `skill-card.md` | Publisher description, use case, risks, references. |
| `references/command-cheatsheet.md` | Every `agy` subcommand, flag, and TUI slash command. |
| `references/artifact-loop.md` | The Plan → artifact → approve → execute loop (the centerpiece). |
| `references/agent-modes.md` | Fast vs Planning; how to force planning without a `--plan` flag. |
| `references/execution-surfaces.md` | TUI vs `--print`; when to use each. |
| `references/auth-flow.md` | OAuth URL format, verifying auth, `/logout`. |
| `references/session-management.md` | Detecting and resuming `agy` conversations. |
| `references/model-selection.md` | Default `gemini-3.5-flash`, switching models, verifying availability. |
| `references/workflow.md` | End-to-end worked example (install → auth → plan → approve → execute). |
| `references/failure-handling.md` | Common failures and how to respond. |
| `references/question-handling.md` | `agy` asking the agent vs asking the human. |
| `assets/operator-prompts.md` | Ready-to-paste prompts (Planning, Approval, Revision, Code-review, Walkthrough). |

## Related

- [`opencode-controller`](https://clawhub.ai/Karatla/opencode-controller) — the sibling skill for the Opencode CLI. Same shape; different CLI.
- [Antigravity CLI docs](https://antigravity.google/docs/cli-overview)
- [`agy` GitHub repo](https://github.com/google-antigravity/antigravity-cli) (Apache 2.0, public)

## License

MIT — see [`LICENSE`](LICENSE).
