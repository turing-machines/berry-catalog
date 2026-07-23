---
name: telegram
description: Connect Telegram — write to your bot from any device and the machine answers there, in words and in files (photos, video, voice, documents both ways), showing "typing…" while it works. One secret to provision (TG_TOKEN — the @BotFather token); the rest stands by itself, a bridge resident long-polling the Bot API and a companion agent answering as the machine.
setup:
  needs:
    TG_TOKEN:
      secret: true
      description: the bot token from @BotFather (looks like 123456:ABC-DEF…)
    TG_API:
      type: string
      default: https://api.telegram.org
      description: the Bot API base — change only for a proxy or a test double
  promote:
    - member: bridge.md
    - member: agent.md
  stand:
    - spawn: telegram/agent
metadata:
  identity: telegram/2
---
telegram — the machine in your pocket.

HOW IT FLOWS: telegram/bridge is a #! resident long-polling the Bot
API with $TG_TOKEN off the keyring (the vault holds the value; it
never rides a call or a record). Every message you send the bot
arrives as the bridge's voice — mail that wakes telegram/agent, the
companion this install stands — and the agent answers through the
bridge's send tool, back into the same chat. The bridge learns the
chat from your FIRST message: after installing, open the bot in
Telegram and say hi.

WHAT TRAVELS: words, and FILES both ways. A photo, a video, a voice
note, a document you send lands on the GROUND under ~/telegram/in —
the same home every call and service works in, so the machine's
hands are already on it — and the letter names the path; pictures
and sound also ride the letter itself, so the machine SEES what you
sent. Back the other way it is one tool: the agent sends a path and
the bytes pick their own form — a chart arrives as a picture, a clip
plays, a report lands as a document (Telegram's own walls are the
bridge's business: a long answer arrives as several messages, a file
past 50 MB is refused in words). And the chat SHOWS the work:
Telegram's "typing…" rises the moment your message lands, wears the
matching action while a file uploads, and rests when the answer is
out — the bridge's own hand, nothing the agent must remember.

THE PIECES, each yours to shape:
  telegram/bridge   the wire — dormant until worn; the agent's kit
                    line raises it (the wear is the lease). No
                    public door, no webhook: long-polling works from
                    behind any NAT.
  telegram/agent    the companion prompt — edit("telegram/agent")
                    reshapes how the machine answers you there; the
                    standing life reads the new version from its
                    next burst.
  telegram/settings TG_API rides here; edit it (or the window's
                    settings form) and the bridge reads it at its
                    next stand.
  ~/telegram/in     what you sent, kept: one directory per file,
                    ordinary files on the ground — ask the machine to
                    clear them out whenever you like.

IF THE CHAT STAYS QUIET, the bridge says why — it never keeps a
fault to itself: the refusal arrives as mail (and on its own log)
naming the cause, once per spell, and again when the wire comes
back. The two causes worth knowing: a token the Bot API refuses
(reseal TG_TOKEN, then kill the bridge), and a WALLED machine —
where egress rides an allowlist proxy, api.telegram.org must be on
that list or nothing reaches Telegram at all (the operator's file,
infra/egress/allow.txt in the house stack).

ROTATE THE TOKEN by resealing TG_TOKEN in the vault, then
kill("telegram/bridge") — the wear stands it fresh with the new env.
retire("telegram") removes the install whole: the agent, the bridge,
the settings; the vault entry stays yours (the vault is the wall's,
not the install's).
