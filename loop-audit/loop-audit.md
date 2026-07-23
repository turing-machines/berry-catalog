---
name: loop-audit
description: The loop-audit volume — a reviewer of the seams between the stochastic author and the deterministic substrate: missing funnels, missing oracles, missing ratchets, misplaced rigidity. One install stands its names; spawn "loop-audit/reviewer" over a failure transcript or a seam.
setup:
  promote:
    - member: reviewer.md
metadata:
  identity: loop-audit/2
---
loop-audit — the health of the loops, as software on the machine.

from: the house catalog, volume folder loop-audit/
edition: loop-audit/1

The premise: a stochastic author over a deterministic substrate
fails in FOUR mechanical ways, and none of them lives in a single
file — they live in the loop. This volume's reviewer walks one seam
(a door, an organ, a skill, a failure transcript) and sorts what it
finds into the four columns: where a distribution should collapse
but doesn't (funnels), where a mistake dies silently (oracles),
where progress can slide back (ratchets), where a rule fights the
flow instead of catching it (rigidity).

members:
  loop-audit/reviewer   the auditor prompt: spawn("loop-audit/reviewer",
                        "<what to audit — a transcript, a door, a
                        skill>") for one pass — the four-column report
                        arrives as your mail; or stand it on the clock
                        over fresh journals — schedule(spawn:
                        "loop-audit/reviewer", text: "audit the
                        freshest failure in the journal", every:
                        "1d"). It proposes mechanisms, never
                        prohibitions; the user disposes.

The install is ONE act — the setup head above declares it: the member
stands promoted, the index binds last, and retire("loop-audit")
removes the install whole.
