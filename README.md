# berry catalog

The house catalog: volumes for berry — one folder per volume, ONE
shape: a tree. A folder holds an INDEX (`SKILL.md`, the Agent Skills
standard verbatim — agentskills.io — or `<name>.md`, the machine's
record dialect) plus whatever rides beside it (`scripts/`,
`references/`, member records). The index head may declare a
`setup:` — the volume's install plan, and the whole install is one
deterministic act of the machine:

```yaml
setup:
  deps:                 # other catalog volumes, "name" or "name/N"
    - browser           # (N — the minimum source identity; an
  needs:                #  install may raise a dep, never lower it)
    TG_TOKEN:
      secret: true      # resolves in the vault BY NAME — the value
      description: …    # never rides a call or a record
    GREETING:
      type: string      # a plain need — lives in "<volume>/settings",
      default: hi       # edited without a reinstall
  promote:              # members that stand on the name plane —
    - member: shelf.md  # "<volume>/<leaf>" by default (capture-free);
    - member: cli.md    # an explicit bare `as:` refuses on capture
      as: mycli
  stand:                # prose prompts born as standing agents;
    - spawn: curator/shelf   # a #! launcher rises on demand instead
```

No setup and a described `SKILL.md` is the convention: adopt + the
skill promoted as `<volume>/skill` — a one-tap install. There is NO
exec at install — a `#!` body prepares its own ground at first
stand, which is exactly what lets a restore replay every install
whole. `retire(<volume>)` removes an install whole: names, settings,
stood lives; deps stay while another install still declares them.

Curation is the wall: every file is read before it enters the
catalog (an adopted text is followed, an adopted `#!` runs), the
license permits redistribution, script dependencies are pinned.

Volumes:

- `claude-code/` — house-authored, Apache-2.0. Installing Anthropic's
  Claude Code CLI as a standing resident: the reboot-proof install
  path, the model wire read from the env (never a vendor sign-in),
  the worker seat for its permission-bypass flag, the stdin-loop
  stand, and the product's other faces (one-shot print runs, MCP
  server). Flags verified against claude-code 2.1.214.
- `curator/` — the shelf curator: a scheduled registry reviewer that
  proposes retirements and keeps indexes honest; installs to
  `curator/shelf`.
- `loop-audit/` — the loop auditor: reviews one seam between the
  stochastic author and the deterministic substrate for missing
  funnels, missing oracles, missing ratchets and misplaced rigidity;
  installs to `loop-audit/reviewer`.
- `telegram/` — house-authored. Connect Telegram: a bridge resident
  long-polling the Bot API (TG_TOKEN — the one secret, sealed to the
  vault; no webhook, no public door) and a companion agent answering
  the user's messages there; installs to `telegram/bridge` +
  `telegram/agent`, the agent stands at install. Files travel both
  ways — an attachment lands on the ground and rides the letter for
  the machine to see, an answer sends a path back as the kind its
  bytes are — and the chat wears Telegram's own "typing…" while the
  work runs.
- `skill-creator/` — Apache-2.0, from anthropics/skills. Create, test
  and improve Agent Skills: the drafting flow, eval scripts, a spec
  validator (`scripts/quick_validate.py`, needs Python 3 + PyYAML), a
  packager and a review viewer. Its description-optimization scripts
  shell out to the `claude` CLI — where that CLI is absent the skill
  itself says to skip them.
