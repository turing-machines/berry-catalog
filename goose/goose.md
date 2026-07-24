---
name: goose
description: Install Block's goose agent CLI and stand it as a DRIVEN foreign agent over ACP — the machine drives its prompt turns on the ground and serves its hands (files, terminal, permission), delegated as goose/prompt. Use when asked to set up, install or stand goose as an agent on this machine.
license: Apache-2.0
compatibility: Needs network egress for the installer and bash in the jail; goose reads its provider from the machine's model wire (the env) or its own config.
setup:
  needs:
    GOOSE_PROVIDER:
      type: string
      default: anthropic
      enum: [anthropic, openai, google, groq, ollama]
      description: which provider goose dials — it reads the machine's model wire from the env; change only for a non-default provider
  stand:
    - spawn: goose
      protocol: acp
      command: 'B="$HOME/.local/bin/goose"; [ -x "$B" ] || curl -fsSL https://github.com/block/goose/releases/download/stable/download_cli.sh | CONFIGURE=false bash 1>&2; exec "$B" acp'
metadata:
  identity: goose/1
---
goose is a foreign coding agent — on this machine it stands as a
DRIVEN AGENT over ACP (lookup("resident-skill") § THE DRIVEN WIRE
holds the general law: the stand, the drive, the served hands, the
proof; this volume holds only goose's own knobs).

THE WIRE — goose reads its provider from the env, and the machine
bakes its OWN model socket into every call and service (the self-wire:
base URL + key under the common conventions). There is no vendor
sign-in to hunt and no per-run --model choice: point goose at the
machine's wire and the model answers. WHICH model is the machine's
identity, never goose's to pick. Read `goose configure` and
`goose --help` WHOLE before standing — goose's provider knobs drift
between versions, and its help is the contract this volume's names
were verified against.

INSTALL — reboot-proof or it dies at the next boot: what lands outside
/work reverts at a jail reboot while the journaled stand re-runs
verbatim, so the binary must live ON THE GROUND. goose's own installer
lands in the home:

    curl -fsSL https://github.com/block/goose/releases/download/stable/download_cli.sh | bash

(verify goose's CURRENT install line from its README — installers
drift; the point is: on the ground, not a container-layer global.)
Then `goose --version`, and `goose acp --help` to confirm the ACP
subcommand this build spells.

THE STAND — goose ships an ACP subcommand; stand it as the driven
wire, bare, no seat, no flags:

    spawn("goose", command: "goose acp", protocol: "acp")

THE PROOF — one delegated task through its own bridge tool. Wear
"goose/*" in your kit, then:

    goose__prompt {"prompt": "reply with the single word: ok"}

— the reply is goose's own, run on your ground. Everything else — the
served hands (its files are your ground, its shell your computer, its
permission answered at your door), the reboot law, the one-call proof
discipline — is the general law in resident-skill; this volume adds
only the install and the acp command.
