# skill-antigravity

OpenClaw skill for driving **Google Antigravity CLI** (`agy`) — Google's agentic development platform (Nov 2025, VS Code fork + agent-first Mission Control).

This is an **operator-prompt skill**: pure markdown, no Python, no dependencies. It teaches an openclaw agent how to run `agy` headless (`-p`) through a structured **Plan → approve → execute** loop, capture `agy`'s markdown proposals, surface them to the human, and only then run the build.

**This is NOT a code library.** There are no Python modules to import, no API to call. The skill is a markdown contract the consuming agent follows.

## What this skill does

- Wraps the `agy` CLI's real surface: `-p` (headless print) for agent-driven work, TUI for user-driven work, `-c` / `--conversation` for resumption.
- Drives the **Plan → approve → execute** loop:
  1. **Plan** — the agent sends the task to `agy -p` with a planning prompt that asks for a markdown proposal. No code is written.
  2. **Approve** — the agent surfaces the proposal to the human **verbatim** and waits for the explicit word **Approve** (or a clear affirmative, or a comment for revision).
  3. **Build** — once approved, the agent resumes the same conversation with `agy -p -c` and the build prompt; `agy` runs the plan, writes files, and reports results.
- Manages session reuse (`agy -c`, `agy --conversation <id>`), model selection (default from `settings.json`), and OAuth handoff.
- Forces a planning pass on every non-trivial task by prompt phrasing — `agy` has no `--plan` flag, so the planning prompt is the gate.

## Install

The skill is a directory of markdown files consumed by an openclaw agent. To use it:

```bash
# Clone the skill
git clone https://github.com/A-I-M-S/skill-antigravity.git
cd skill-antigravity

# The skill itself needs no install. The CLI it drives does:
curl -fsSL https://antigravity.google/cli/install.sh | bash
source ~/.bashrc
which agy   # ~/.local/bin/agy
```

## Quick start

```bash
# 1. Verify the binary
which agy          # expect ~/.local/bin/agy
agy --version      # validated against 1.0.6

# 2. Authenticate (one-time, browser OAuth via Antigravity)
agy
# → CLI opens browser, user signs in
# → settings.json is populated with model and project

# 3. Verify auth
agy models         # returns the model list = signed in

# 4. Inspect default settings
cat ~/.gemini/antigravity-cli/settings.json
# { "model": "Gemini 3.1 Pro (High)", "gcp": {"project": "...", "location": "global"}, "enableTelemetry": false }

# 5. Read the procedure
# The agent reads SKILL.md and follows the three-step loop.
# See references/workflow.md for the end-to-end walkthrough.
```

## Layout

| File | Purpose |
|------|---------|
| `SKILL.md` | Main procedure the agent follows. |
| `skill-card.md` | Publisher description, use case, risks, references. |
| `references/command-cheatsheet.md` | Real `agy` flags, subcommands, and TUI slash commands (validated against v1.0.6). |
| `references/artifact-loop.md` | The Plan → approve → execute loop, the approval-gate rules, the revision round-trip. |
| `references/execution-surfaces.md` | `-p` (headless) vs TUI vs resume; when to use each. |
| `references/auth-flow.md` | OAuth via Antigravity, settings.json, `/logout`. |
| `references/session-management.md` | Detecting and resuming `agy` conversations. |
| `references/model-selection.md` | Default from `settings.json`, switching models, verifying availability. |
| `references/workflow.md` | End-to-end worked example (install → auth → plan → approve → execute). |
| `references/failure-handling.md` | Common failures and how to respond. |
| `references/question-handling.md` | `agy` asking the agent vs asking the human; never inventing "yes" on destructive actions. |
| `assets/operator-prompts.md` | Ready-to-paste prompts (Plan, Approve, Revise, Build, Code-review, Walkthrough, Re-auth). |

## Related

- [`opencode-controller`](https://clawhub.ai/Karatla/opencode-controller) — the sibling skill for the Opencode CLI. Same shape; different CLI.
- [Antigravity CLI docs](https://antigravity.google/docs/cli-overview)
- [`agy` GitHub repo](https://github.com/google-antigravity/antigravity-cli) (Apache 2.0, public)

## License

MIT — see [`LICENSE`](LICENSE).
