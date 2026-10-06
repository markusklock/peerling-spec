---
title: "D-0021: PvP win counter"
type: decision
status: accepted
tags: [gameplay, pvp, multiplayer]
sources:
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-pvp-wins-hex-world.md
  - raw/conversations/2026-10-06-proposals-approved.md
related:
  - wiki/gameplay/pvp-battles.md
  - wiki/tech/player-data.md
updated: 2026-10-06
---

# D-0021: PvP win counter

**Status:** accepted (2026-10-06). Changes the "no rewards" rule in
[pvp-battles § Rewards](../gameplay/pvp-battles.md#rewards).

## Context
PvP battles start at full HP and change nothing afterwards, and they give no
XP. The designer: *"with nothing to lose or gain on PvP it feel meaningless.
If we add at least a counter of battles won you at least get something from
PvP"*. Earlier, PvP had no rewards because a third party couldn't verify the
result.

## Decision
[accepted] Each player has a **counter of PvP battles won**, shown on their
profile. It is a stat only: still no XP or items from PvP.

[accepted] How results are proven and recorded (approved 2026-10-06): both
players sign every battle state and the end of the battle, and each player
records a signed `pvp-result` save-log event:
[pvp-battles § Win record](../gameplay/pvp-battles.md#win-record).

## Consequences
- PvP results must be provable by third parties, so the battle protocol needs
  signed results.
- Win counts can be inflated by fixed battles between friends; with no
  reward attached this is accepted.

## Alternatives considered
- **No PvP rewards at all** (the previous rule): PvP feels pointless.
- **XP or items for wins:** farmable, and would make collusion worth it.
- **A ranking (Elo-style) ladder:** more motivating, but much more exposed to
  collusion and alternate accounts; could come later on top of the counter.
