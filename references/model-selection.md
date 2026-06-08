## Model selection procedure

- **Default per user decision:** `gemini-3.5-flash`.
- Open the model selector in the TUI:
  `/model`
- Switch per-invocation on the CLI:
  `agy --model <name>` or `agy -m <name>`
- Verify the model is actually available (auth required):
  `agy models`

## Available models (as of Antigravity v1.0.6)

- `gemini-3.5-flash` — default, fast, low cost.
- `gemini-3-pro` — higher quality, slower.
- `claude-sonnet-4.5` — Anthropic model exposed by Antigravity.
- `gpt-oss` — open-source GPT variant exposed by Antigravity.

The agent must NOT hard-code a model list. Run `agy models` and use one it returns.

## Auth-based model access

- `agy` uses OAuth (no API key path). If `agy models` returns "Please sign in", the user must run `agy` interactively once to complete browser auth.
- See `references/auth-flow.md` for the OAuth URL format and the verbatim-surfacing rule.

## Switching mid-conversation

- In the TUI: type `/model` and pick from the list. The new model applies to subsequent turns; the conversation history is preserved.
- On the CLI: only `--model <name>` on the next `agy` invocation; it does not mutate an existing TUI session.

Never assume a model is available. Verify with `agy models` before recommending it.
