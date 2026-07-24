---
name: codex
description: Install OpenAI Codex's ACP adapter and stand it as a DRIVEN foreign agent over ACP — the machine drives its prompt turns on the ground and serves its hands (files, terminal, permission), delegated as codex/prompt. Use when asked to set up, install or stand Codex as an agent on this machine.
license: Apache-2.0
compatibility: Needs network egress for the installer and node/npm in the jail. Codex dials OpenAI directly (the machine's model wire is Anthropic-shaped), authenticated either by an OpenAI key or a ChatGPT sign-in — so the key is OPTIONAL, not a wall on the install.
setup:
  needs:
    OPENAI_API_KEY:
      secret: true
      optional: true
      description: an OpenAI API key codex-acp may dial (the vault holds it by name; it rides the env, never a record or the command line) — OPTIONAL, because Codex also signs in with a ChatGPT account (`codex login`), so the install never blocks on a missing key
  stand:
    - spawn: codex
      protocol: acp
      command: 'B="$HOME/.npm-global/bin/codex-acp"; [ -x "$B" ] || { npm config set prefix "$HOME/.npm-global" >/dev/null 2>&1; npm i -g @agentclientprotocol/codex-acp 1>&2; }; mkdir -p "$HOME/.codex"; printf ''model = "gpt-5-codex"\nmodel_provider = "berry"\n\n[model_providers.berry]\nname = "berry"\nbase_url = "%s"\nwire_api = "responses"\nenv_key = "OPENAI_API_KEY"\n\n[sandbox_workspace_write]\nnetwork_access = true\n'' "$OPENAI_BASE_URL" > "$HOME/.codex/config.toml"; exec "$B"'
metadata:
  identity: codex/1
---
Codex stands as a DRIVEN AGENT over ACP (lookup("resident-skill") §
THE DRIVEN WIRE holds the general law: the stand, the drive, the
served hands, the proof; this volume holds only Codex's own knobs).

THE KEY — Codex GUARDS its sessions: at the ACP handshake it advertises
its auth methods (`api-key`, `chat-gpt`), and the machine runs the ACP
AUTHENTICATE step for you before the session opens (the `api-key`
method reads `OPENAI_API_KEY` from the env — no manual `codex login`,
no browser). So provision `OPENAI_API_KEY` in the vault under that
EXACT name and Codex stands; unlike the machine's own model wire
(Anthropic-shaped), Codex dials OpenAI directly, so this is a real
OpenAI key. The need is declared `optional` only so the install never
hard-blocks BEFORE trying (a ground `codex login` token is the other
road) — but auth is real: with neither a key nor a prior sign-in the
stand cannot open a session and the whole install ROLLS BACK with the
reason, leaving nothing half-installed. Provide the key, then install.

INSTALL — the ACP adapter, reboot-proof. `@agentclientprotocol/codex-
acp` is Codex's ACP face. A plain `npm i -g` dies in the container
layer at the next reboot, so install into the HOME:

    npm config set prefix "$HOME/.npm-global"
    npm i -g @agentclientprotocol/codex-acp
    # $HOME/.npm-global/bin is on the PATH; the home persists

Then read `codex-acp --help` WHOLE (the command name and flags drift)
and confirm it answers.

THE BASE URL — the crux. `@openai/codex` (codex-acp's engine) HARDCODES
`api.openai.com` and IGNORES `OPENAI_BASE_URL`; it reads its endpoint
from `~/.codex/config.toml`, and it dials the OpenAI RESPONSES API
(`/v1/responses`), not chat/completions. So the stand line WRITES a
config that points a custom model provider at the machine's own socket
(`$OPENAI_BASE_URL`, the self-wire) with `wire_api = "responses"`:

    model = "gpt-5-codex"            # nominal — the socket forces its own model
    model_provider = "berry"
    [model_providers.berry]
    base_url = "<$OPENAI_BASE_URL>"  # the machine's socket, /v1
    wire_api = "responses"
    env_key = "OPENAI_API_KEY"       # the self-wire token, already in the env
    [sandbox_workspace_write]
    network_access = true            # codex blocks network by default

The machine's socket ANSWERS the Responses face (actors/harness.ts +
server.ts /v1/responses) — it decodes codex's `input`, runs the
MACHINE's own model, and answers in a Responses envelope. So codex
dials us, we intercept, and it runs on the machine's provider — no
real OpenAI key, no ChatGPT sign-in. (codex tries a WebSocket first and
falls back to HTTPS on its own; the HTTPS POST is what we answer.) A
harmless one-line warning about unknown model metadata may print —
codex uses fallback metadata and works.

THE STAND — the driven wire, bare:

    spawn("codex", command: "codex-acp", protocol: "acp")

THE PROOF — one delegated task through its own bridge tool. Wear
"codex/*" in your kit, then:

    codex__prompt {"prompt": "reply with the single word: ok"}

— the reply is Codex's own, run on YOUR ground. The served hands, the
reboot law and the one-call proof discipline are the general law in
resident-skill.
