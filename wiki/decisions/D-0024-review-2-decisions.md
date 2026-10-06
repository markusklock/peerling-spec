---
title: "D-0024: Determinism, verification and edge-case rules from the second review"
type: decision
status: accepted
tags: [tech, verification, gameplay, world]
sources:
  - raw/conversations/2026-10-06-review-2-fixes.md
  - raw/conversations/2026-10-06-review-2-decisions.md
related:
  - wiki/world/procedural-generation.md
  - wiki/tech/player-data.md
  - wiki/tech/data-formats.md
  - wiki/gameplay/pvp-battles.md
updated: 2026-10-06
---

# D-0024: Determinism, verification and edge-case rules from the second review

**Status:** accepted (2026-10-06). Changes parts of D-0013 (transfer conflict
resolution), D-0017 (world generation), D-0021 (PvP win proof) and D-0022
(guardians, Peerling of the Day, first finds).

## Context
A second full review found rules that two honest clients could compute
differently (world generation, randomness source, client-derived epoch
records, the novelty weight), weak spots in trades and PvP results, and
undefined edge cases.

## Decision
[accepted]
1. Tile kind, height and biome use integer maths only, from a fixed world
   seed per generator version ([procedural-generation § Deterministic world maths](../world/procedural-generation.md#deterministic-world-maths)).
2. Randomness comes from drand **quicknet** with an exact round per epoch; no
   server-made fallback ([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)).
3. A client-derived epoch record is valid only if it copies a server-signed
   record at least 2 epochs older; for day and week records the signed record
   wins.
4. "Seen" for the novelty weight is defined exactly from save events
   ([encounters § Selection](../gameplay/encounters.md#selection)); catch
   evidence carries the Peerling of the Day record.
5. Identical copies of a transfer never conflict; a double trade is two
   different transfers after the same one, and the lower envelope CID wins;
   no cancelling a trade after signatures are exchanged
   ([trading](../gameplay/trading.md)).
6. Recoil isn't applied when the hit ended the battle; PvP draws after 200
   turns ([battle](../gameplay/battle.md#ending-a-battle)).
7. A loser-signed win beats a forfeit claim; conflicting forfeit claims cancel;
   the in-game profile counts only verified wins
   ([pvp-battles § Win record](../gameplay/pvp-battles.md#win-record)).
8. Shrine: one open job or credit at a time, failed jobs give a credit, a
   status endpoint; unverified offering levels are an accepted gap
   ([creation-shrine](../gameplay/creation-shrine.md)).
9. The first-finder credit is final once announced.
10. Guardians: XP only for a win; smaller teams when few species exist; teams
    fixed for the week even if a species is delisted.
11. A delisted Peerling of the Day loses its boost; the pedestal stays empty.
12. The team always keeps a non-fainted Peerling outside battle.
13. Hiding your follower hides it from others too.
14. A 60 m border ocean; guardian sites on walkable land; the spawn hub is the
    hexagon's landmark and rest point.

## Consequences
- Catch and badge verification gives the same result on every client.
- New fields: `dayRecord`, checkpoint `state`, `battle-result` `candidate` and
  `species`; new endpoint `GET /v1/shrine`; new results `draw`.
- Two new accepted gaps: base-record choice during outages, and unverified
  shrine offering levels.

## Alternatives considered
Each item's alternatives were listed in the review; the main ones were
floating-point world generation with tolerance checks (rejected: tile kinds
can't be "almost" foliage), a server-generated randomness fallback (rejected:
unverifiable), a fixed level for shrine creations, and letting a later-synced
catch take the first-finder credit.
