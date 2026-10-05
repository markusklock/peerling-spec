# 2026-10-05 — Peer save backups through profile inspection

Kind: design conversation. The LLM had proposed keeping saves available via
(A) a save snapshot in the backup file and (B) community mirrors following save
logs.

## Designer's statement (verbatim)

> No, I do not think community mirrors will be a thing so we should not count on them. Is there another approach? When a player is close to another player can we make it possible to inspect the other players team, and doing so also fetches the other players full save file, creating sporadic backups of them?
> Is this data persistent if the player close his browser and restart his computer? (given that the browser cache is NOT cleared)

## Summary of what was decided

- Community mirrors are not expected; the design must not count on them.
- Proposed by the designer: inspecting a nearby player's team also fetches their
  full save, so players hold sporadic backups of each other's saves. The LLM
  wrote this up in detail as a proposal (Q-043).
- The designer asked whether such data survives closing the browser and
  restarting the computer; answered (yes, with caveats) and filed as a
  proposed storage requirement.
