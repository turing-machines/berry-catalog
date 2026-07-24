---
name: claude-code
description: Install Claude Code's ACP adapter and stand it as a DRIVEN foreign agent over ACP — the machine drives its prompt turns on the ground and serves its hands (files, terminal, permission), delegated as claude-code/prompt. The model wire read from the env (never a vendor sign-in). Use when asked to set up, install or stand Claude Code (or "claude") as an agent on this machine.
license: Apache-2.0
compatibility: Needs network egress for the installer, node/npm in the jail, and the machine's model wire in the env (ANTHROPIC_BASE_URL + ANTHROPIC_API_KEY).
setup:
  promote:
    - member: SKILL.md
  stand:
    - spawn: claude-code
      protocol: acp
      command: 'B="$HOME/.npm-global/bin/claude-agent-acp"; [ -x "$B" ] || { npm config set prefix "$HOME/.npm-global" >/dev/null 2>&1; npm i -g @agentclientprotocol/claude-agent-acp 1>&2; }; exec "$B"'
metadata:
  identity: claude-code/2
  verified-against: "@agentclientprotocol/claude-agent-acp (Claude Code 2.x)"
---
Claude Code stands as a DRIVEN AGENT over ACP — the robust road
(lookup("resident-skill") § THE DRIVEN WIRE holds the general law;
this volume holds the product's own knobs). The OLD stdin-loop stand —
a shell loop over `claude -p`, the worker seat, `--dangerously-skip-
permissions` — is GONE: over ACP the agent asks the MACHINE for
permission and the machine answers at its own door, so no seat, no
bypass flag, no loop, none of the dash-quoting hazards that stand ate.

THE WIRE IS ALREADY THERE — settle this BEFORE anything else. The
adapter reads `ANTHROPIC_BASE_URL` + `ANTHROPIC_API_KEY` from the env,
and the machine bakes BOTH into every call and every service. There is
nothing to provision: no `claude /login`, no `setup-token`, no OAuth,
no key hunted in the keyring, no question to the user about accounts.
The machine's socket answers the Anthropic wire face and forces its
OWN model whatever name the CLI asks — so no `--model` fuss either. A
run that stops to wonder "but where is the key?" is the observed
failure mode: the key EXISTS, the adapter is already looking at it.

INSTALL — the ACP adapter, reboot-proof. `@agentclientprotocol/claude-
agent-acp` is the ACP face of Claude Code. A plain `npm i -g` lands in
the CONTAINER LAYER and dies at the next jail reboot (the journaled
stand then re-runs a command whose binary is gone); install into the
HOME so the standing survives:

    npm config set prefix "$HOME/.npm-global"
    npm i -g @agentclientprotocol/claude-agent-acp
    # $HOME/.npm-global/bin is on the PATH; the home persists

Then read `claude-agent-acp --help` WHOLE (the exact command name and
flags drift between versions; the aliases claude-code-acp / claude-code
may also stand) and confirm it answers. If this build of the adapter
needs the `claude` CLI beside it, install THAT on the ground too — the
native installer lands it in the home:

    curl -fsSL https://claude.ai/install.sh | bash   # -> ~/.local/bin/claude

INSTALL, not authorship: never hand-roll a wrapper — the published
adapter IS the ACP face; a stand-in is not the agent the user asked
for (volume-skill holds the rule).

THE STAND — INSTALL STANDS IT FOR YOU. This volume's `setup.stand`
carries the driven wire, so installing claude-code stands the agent at
once: it appears in the sidebar as a DRIVEN AGENT, journaled and
restored across reboots, no further step. The stand's `command` is a
SELF-INSTALLING launcher — first run lands the adapter into the home
(the npm lines above, once), every run `exec`s it as the ACP wire — so
there is nothing to run by hand. The bare form, for the record:

    spawn("claude-code", command: "claude-agent-acp", protocol: "acp")

THE PROOF — one delegated task through its own bridge tool. Wear
"claude-code/*" in your kit, then:

    claude-code__prompt {"prompt": "reply with the single word: ok"}

— the reply is Claude Code's own, run on YOUR ground (its files are
your ground, its shell your computer, its permission answered at your
door). The reboot law and the one-call proof discipline are the
general law in resident-skill.

THE OTHER FACES stay the ASK's to pick, never a menu back at the user.
"Set up Claude Code as an agent" is this DRIVEN face — done above. A
one-shot transform inside your OWN work is a mortal `claude -p` run (no
stand at all). `claude mcp serve` is the MCP-SERVER face — tools for a
kit (THE FRAMED WIRE), a different ask. The bare `claude` TUI is a
full-screen face and cannot stand as a service (no keyboard in a pipe).
Standing one where a driven agent was asked leaves nobody to delegate
to.
