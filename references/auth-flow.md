## OAuth auth flow

`agy` uses OAuth via Google's authorization server. There is no API-key path. The token is stored in the system keyring.

## First-run flow

On the first `agy` invocation, the CLI prints a URL and waits for the user to complete browser auth. The URL looks like:

```
Authentication required. Please visit the URL to log in:
  https://accounts.google.com/o/oauth2/auth?access_type=offline&client_id=...&...
```

- The URL is valid for ~30 seconds. If the user does not complete it in time, re-invoke `agy` to regenerate.
- Locally, `agy` opens the default browser to that URL.
- On remote/SSH, `agy` only prints the URL — the user must copy it to a local browser.
- The OAuth token lands in the system keyring (GNOME Keyring, macOS Keychain, Windows Credential Manager, or `secret-tool` on headless Linux).

## Verifying auth

- `agy models` — returns the model list when signed in.
- `agy models` — returns "Please sign in" when not.

The agent must run `agy models` after install and report the result to the user. Do not assume auth.

## Surfacing the URL — verbatim rule

The agent must surface the auth URL **verbatim** to the user:

- Never paraphrase. Never trim query parameters. Never round-trip through a shortener.
- Output the URL inside a fenced code block, on its own line, exactly as `agy` printed it.
- The closing line must be: *"Open the URL in a browser, complete the sign-in, then reply 'done' so I can continue."*

## Sign out

- Open `agy`, type `/logout`. The token is cleared from the system keyring.
- The agent should document this in pre-flight: "`/logout` clears the OAuth token."

## Headless / no-browser environments

If the user is on a remote box with no browser and no X11 forward, the URL is the only path — the user must copy it to a local machine. The agent must not attempt to launch a browser on the remote host. If the user cannot reach `accounts.google.com` from the remote, they must run `agy` on a local machine and resume the conversation on the remote with `agy --conversation <id>` (the conversation id is portable across machines via the `projects.json` cache).

## Re-auth

- If `agy models` returns "Please sign in" mid-task, the token has been cleared or expired.
- Re-run `agy` interactively, complete browser auth, then resume the conversation with `agy -c`.
- The agent must stop and ask the user to confirm re-auth before continuing.
