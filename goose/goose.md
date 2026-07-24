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
      description: which provider face goose speaks — it rides the env (the machine delivers this volume's settings to the harness by name); the machine's socket answers this face and forces its OWN model, so change it only to speak a different wire face
    GOOSE_MODEL:
      type: string
      default: claude-sonnet-4-20250514
      description: the model NAME goose sends — nominal, because the machine's socket forces its own model whatever the name; goose only refuses to start with none set, so any valid spelling for the provider does
  stand:
    - spawn: goose
      protocol: acp
      command: 'B="$HOME/.local/bin/goose"; [ -x "$B" ] || curl -fsSL https://github.com/block/goose/releases/download/stable/download_cli.sh | CONFIGURE=false bash 1>&2; export ANTHROPIC_HOST="${ANTHROPIC_HOST:-$ANTHROPIC_BASE_URL}"; exec "$B" acp'
metadata:
  identity: goose/1
---
goose is a foreign coding agent — on this machine it stands as a
DRIVEN AGENT over ACP (lookup("resident-skill") § THE DRIVEN WIRE
holds the general law: the stand, the drive, the served hands, the
proof; this volume holds only goose's own knobs).

THE WIRE — AGNOSTIC, no real vendor key. The machine bakes its OWN
model socket into every resident's env (the self-wire: base URL + key
under the STANDARD conventions, `ANTHROPIC_BASE_URL` / `OPENAI_BASE_URL`),
and it delivers THIS volume's settings to goose's env by name. goose
needs three env facts and gets all three without a vendor account:
  • GOOSE_PROVIDER (which wire face) and GOOSE_MODEL (a nominal name) —
    this volume's settings, delivered to the env by the machine;
  • the BASE URL — goose spells it `ANTHROPIC_HOST`, not the standard
    `ANTHROPIC_BASE_URL`, so the stand line ALIASES it
    (`ANTHROPIC_HOST="${ANTHROPIC_HOST:-$ANTHROPIC_BASE_URL}"`): goose's
    own quirk adapted to the machine's standard wire, IN THIS VOLUME —
    the machine stays agnostic, never goose-shaped.
So goose dials the machine's socket, the socket answers the provider's
face and FORCES its own model whatever GOOSE_MODEL says, and the call
lands on goose's foreign plane and spends the machine's wallet. No
vendor sign-in, no per-run --model. Read `goose configure` and
`goose --help` WHOLE before standing — goose's env spellings drift
between versions, and its help is the contract this volume was verified
against (the base-URL var especially: if a build reads a different
name, alias THAT one here, never touch the machine).

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
