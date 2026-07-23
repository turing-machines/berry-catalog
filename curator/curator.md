---
name: curator
description: The shelf curator volume — a scheduled reviewer of the registry that proposes retirements and keeps indexes honest. One install stands its names; spawn or schedule "curator/shelf".
setup:
  promote:
    - member: shelf.md
metadata:
  identity: curator/2
---
curator — the shelf's own maintenance, as software on the machine.

from: the house catalog, volume folder curator/

members:
  curator/shelf   the reviewer prompt: spawn("curator/shelf", "review
                  the shelf") for one pass, or stand it on the clock —
                  schedule(spawn: "curator/shelf", text: "review the
                  shelf", every: "1d") — and each report arrives as
                  mail: findings and proposed retirements, the user as
                  court of appeal.

The install is ONE act — the setup head above declares it: the member
stands promoted, the index binds last, and retire("curator") removes
the install whole. A machine that adopted the old two-record shape
keeps answering: every version stays pinned.
