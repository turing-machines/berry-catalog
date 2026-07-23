---
name: claude-code
description: Install Anthropic's Claude Code CLI on this machine and stand it as a conversation service — the reboot-proof install path, the model wire read from the env (never a vendor sign-in), the worker seat for its permission-bypass flag, the stdin-loop stand, and its other faces (one-shot print runs, MCP server). Use when asked to set up, install or stand Claude Code (or "claude") as an agent, service or resident.
license: Apache-2.0
compatibility: Needs network egress for the installer, bash in the jail, and the machine's model wire in the env (ANTHROPIC_BASE_URL + ANTHROPIC_API_KEY).
metadata:
  identity: claude-code/1
  verified-against: claude-code 2.1.214
---
Claude Code is a foreign agent CLI — on this machine that means ONE
road (lookup("resident-skill") holds the general law, this volume holds
the product's own knobs): install on the ground, one mortal run to
prove the seat, stand the stdin loop as a service, prove it through
its own ear. Everything below is the straight walk of that road; every
trap named here has eaten real bursts.

THE WIRE IS ALREADY THERE — settle this BEFORE touching auth. Claude
Code reads `ANTHROPIC_BASE_URL` and `ANTHROPIC_API_KEY` from the env
natively, and the machine bakes BOTH into every call and every
service. There is nothing to provision: no `claude /login`, no
`claude setup-token`, no OAuth, no key hunted in the keyring, no
question to the user about accounts. The machine's socket answers the
Anthropic wire face and forces its OWN model whatever model name the
CLI asks for — so no `--model` fuss either. A run that stops to
wonder "but where is the key?" is the observed failure mode: the key
EXISTS, the program is already looking at it.

INSTALL — reboot-proof or it dies at the next boot. The law of the
jail: what lands outside /work reverts at a jail reboot, while the
journaled service command is re-stood VERBATIM — a stand whose binary
lived in the container layer dies at every boot. So install INTO THE
HOME (the home is on the ground and persists), which the native
installer does by itself:

    curl -fsSL https://claude.ai/install.sh | bash

Pipe to `bash`, not `sh` — the installer refuses plain sh. It lands
the binary at `~/.local/bin/claude`, which is already on every call's
PATH. Verify with `claude --version`. Do NOT `npm install -g` — the
global prefix sits in the container layer and the standing dies at
the first reboot.

Then read `claude --help` WHOLE before standing anything: flags drift
between versions, and the help is the contract this volume's flag
names were verified against.

THE SEAT — Claude Code refuses `--dangerously-skip-permissions` under
root, and the flag is required headless (no TTY exists to approve
tool calls; without it a print run just denies its own tools). The
machine's standing answer is the unprivileged `worker` seat (same
home, made by the machine at every boot — never useradd your own).
One mortal run FIRST, before any stand:

    printf 'say READY\n' | su -s /bin/bash worker -c \
      'claude -p --dangerously-skip-permissions --bare'

`-s /bin/bash`, not /bin/sh — the jail's sh is dash, and dash eats
herestrings and bash-isms silently. Feed the prompt through a PIPE,
never `<<<`. `--bare` keeps the run minimal (no auto-memory, no hook
chatter — the right posture for a programmatic seat; if an older
build's help lacks the flag, drop it). A plain-text answer back means
wire, seat and binary all stand — only then stand the service.

THE STAND — the conversation service is a LOOP OVER STDIN, one typed
line = one headless run, the answer out on stdout as plain speech:

    spawn("claude-code", command: "su -s /bin/bash worker -c 'while IFS= read -r line; do printf \"%s\\n\" \"$line\" | claude -p --dangerously-skip-permissions --bare; done'")

The pipe inside the loop matters twice: it hands claude ONE line as
its whole stdin (a bare `claude -p` inside the loop would swallow the
loop's later lines), and it survives dash-quoting hazards. Keep the
output DEFAULT TEXT — `--output-format json` wraps the answer in a
session/cost envelope, which is parser food, not speech; a window
reading the resident's voice shows the envelope, not the words.

Each line is a FRESH run — no memory between posts. When the ask
wants a conversation that remembers, add `-c` (continue the latest
session on this home) inside the loop; the first-ever line has no
session to continue, so seed one with the mortal run above first.
Other knobs, only when the ask names the need: `--system-prompt`
REPLACES the default prompt (own what it sees), `--append-system-prompt`
appends; `--no-session-persistence` keeps prompts out of on-disk
session files.

Anything the stand leans on lives ON THE GROUND: a launcher script, a
log, a workspace. /tmp reverts at reboot and the re-stood service
dies reaching for it. And the seat is unprivileged: a directory you
make as root is root-owned and worker CANNOT write into it — `chown
-R worker <dir>` any workspace you hand it (the home's top level is
already world-writable).

THE PROOF goes through the standing resident's own ear:
post(to: "claude-code", text: "…"), then PARK — the answer arrives as
its stdout voice, as mail, and wakes you (wear "claude-code/*" in
your kit to be woken directly). Never prove the service with a
separate one-shot run beside it: a beside-run proves a different
process.

THE OTHER FACES — the ask picks the face, never a menu back at the
user. "Set up Claude Code as an agent" is the CONVERSATION face: the
stdin loop above, done. A one-shot transform inside your own work is
a MORTAL RUN (the `-p` line, no stand at all). `claude mcp serve`
stands it as an MCP server — tools for a kit, a different ask.
The bare `claude` TUI is a full-screen face and CANNOT stand as a
service (there is no keyboard in a pipe); the web/remote UI answers
"a page a human opens". Standing any of these where a conversation
was asked leaves nobody to post to.
