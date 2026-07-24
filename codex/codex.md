---
name: codex
description: Install OpenAI Codex's ACP adapter and stand it as a DRIVEN foreign agent over ACP — the machine drives its prompt turns on the ground and serves its hands (files, terminal, permission), delegated as codex/prompt. Use when asked to set up, install or stand Codex as an agent on this machine.
license: Apache-2.0
compatibility: Needs network egress for the installer, node/npm in the jail, and an OpenAI key (the machine's model wire is Anthropic-shaped; Codex dials OpenAI directly).
setup:
  needs:
    OPENAI_API_KEY:
      secret: true
      description: the OpenAI API key codex-acp dials (the vault holds it by name; it rides the env, never a record or the command line)
  stand:
    - spawn: codex
      protocol: acp
      command: 'B="$HOME/.npm-global/bin/codex-acp"; [ -x "$B" ] || { npm config set prefix "$HOME/.npm-global" >/dev/null 2>&1; npm i -g @agentclientprotocol/codex-acp 1>&2; }; mkdir -p "$HOME/.codex"; grep -q network_access "$HOME/.codex/config.toml" 2>/dev/null || printf ''sandbox_workspace_write = { network_access = true }\n'' >> "$HOME/.codex/config.toml"; exec "$B"'
metadata:
  identity: codex/1
---
Codex stands as a DRIVEN AGENT over ACP (lookup("resident-skill") §
THE DRIVEN WIRE holds the general law: the stand, the drive, the
served hands, the proof; this volume holds only Codex's own knobs).

THE KEY — codex-acp reads `OPENAI_API_KEY` from the env. Provision it
in the vault under that EXACT name (this volume's need declares it, so
the install asks for it once); the machine rides it into the service's
env by name — never a record, never the command line. Unlike the
machine's own model wire (Anthropic-shaped), Codex dials OpenAI
directly, so this is a real key to provide.

INSTALL — the ACP adapter, reboot-proof. `@agentclientprotocol/codex-
acp` is Codex's ACP face. A plain `npm i -g` dies in the container
layer at the next reboot, so install into the HOME:

    npm config set prefix "$HOME/.npm-global"
    npm i -g @agentclientprotocol/codex-acp
    # $HOME/.npm-global/bin is on the PATH; the home persists

Then read `codex-acp --help` WHOLE (the command name and flags drift)
and confirm it answers.

THE SANDBOX WIRE — Codex sandboxes its own workspace and blocks
network by default, so it must be told it may reach out (to the model,
and to anything a task needs). Set its config on the GROUND, in
Codex's own config file (`~/.codex/config.toml` — read its docs for
the current path), so the stand line stays clean of nested-quote
hazards:

    sandbox_workspace_write = { network_access = true }

(If the build only takes it via the `CODEX_CONFIG` env, that non-secret
config rides the command line — `CODEX_CONFIG='{"sandbox_workspace_write":
{"network_access":true}}' codex-acp` — but mind the quoting under
/bin/sh; the config file is the calmer road.)

THE STAND — the driven wire, bare:

    spawn("codex", command: "codex-acp", protocol: "acp")

THE PROOF — one delegated task through its own bridge tool. Wear
"codex/*" in your kit, then:

    codex__prompt {"prompt": "reply with the single word: ok"}

— the reply is Codex's own, run on YOUR ground. The served hands, the
reboot law and the one-call proof discipline are the general law in
resident-skill.
