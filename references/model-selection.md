## Model selection procedure

## Default model

The default is whatever the user picked at first run, stored in `~/.gemini/antigravity-cli/settings.json` → `model`. As of v1.0.6, the most common choice is `Gemini 3.1 Pro (High)`.

```bash
grep '"model"' ~/.gemini/antigravity-cli/settings.json
```

The agent reads this in pre-flight and uses the value as the default. The user can override per-invocation with `--model`.

## Exact model strings (v1.0.6)

`agy models` returns the canonical list, with effort suffix in parentheses. As of v1.0.6:

```text
Gemini 3.1 Pro (Low)
Gemini 3.1 Pro (High)
Gemini 3.5 Flash (Low)
Gemini 3.5 Flash (Medium)
Gemini 3.5 Flash (High)
Gemini 3 Flash
```

The exact string — including the parenthetical effort suffix — is what you pass to `--model` and what appears in `settings.json`. There is no `gemini-3.1-pro-high` slug; you must use the string from `agy models`.

## Listing available models

```bash
agy models
```

This is the source of truth. The agent must **not** hard-code a model list. Run `agy models` and use one it returns. If the user asked for a model that is not listed, surface the `agy models` output verbatim and ask the user to pick one.

## Switching model

### Per-invocation (recommended)

```bash
agy -p "<prompt>" --model "Gemini 3.1 Pro (High)" --sandbox
```

The `--model` flag sets the model for the current invocation. It does not mutate `settings.json`. To make a change persistent, edit `settings.json` directly or change it in the TUI via `/model`.

### In the TUI (user-driven)

The user can type `/model` in the TUI and pick from the list. The new model applies to subsequent turns; the conversation history is preserved.

### Persistently in settings.json

Edit `~/.gemini/antigravity-cli/settings.json` directly. Example:

```json
{
  "model": "Gemini 3.5 Flash (Medium)",
  ...
}
```

There is **no CLI flag** to set the default model. There is **no `agy config` subcommand**. The file is the source of truth.

## Auth-based model access

`agy` uses OAuth (no API-key path). If `agy models` returns a token error, the user must run `agy` interactively to complete browser auth. See `references/auth-flow.md`.

## Switching mid-conversation

- In the TUI: `/model` picks for subsequent turns; conversation history is preserved.
- On the CLI: only `--model` on the next `agy` invocation; it does not mutate an existing TUI session.
- Across `-c` resumes: the `--model` flag overrides per-invocation. If omitted, the conversation continues with whatever model the conversation was started with (or the value in `settings.json` at resume time — verify by running with `--model` explicitly to be safe).

## Cost and quality guidance (informational, not enforced)

- `Gemini 3 Flash` — cheapest, fastest. One-line edits, throwaway scripts.
- `Gemini 3.5 Flash (Low/Medium/High)` — fast and cheap. Multi-step work where latency matters.
- `Gemini 3.1 Pro (Low)` — higher quality, lower cost than High. Most multi-step coding work.
- `Gemini 3.1 Pro (High)` — highest quality. Refactors, security-sensitive changes, code review.

The user picks. The agent surfaces the `agy models` list and asks.
