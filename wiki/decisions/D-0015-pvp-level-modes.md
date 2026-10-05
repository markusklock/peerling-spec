---
title: "D-0015: PvP level modes: Fair or Real levels"
type: decision
status: accepted
tags: [gameplay, pvp, battle]
sources:
  - raw/conversations/2026-10-05-pvp-level-modes.md
related:
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/battle.md
  - wiki/decisions/D-0009-player-data-on-orbitdb.md
updated: 2026-10-05
---

# D-0015: PvP level modes: Fair or Real levels

**Status:** accepted (2026-10-05). Changes the "PvP uses level 50" rule of
[D-0009](D-0009-player-data-on-orbitdb.md) from the only mode to the default.

## Context
PvP set every Peerling to level 50 because levels can't be verified (XP from
wild battles isn't replayed) and to keep battles fair for newcomers. The
downside: levelling up meant nothing in PvP.

## Decision
[accepted] When challenging another player, the challenger picks a mode, and
the other player sees it before accepting:
- **Fair** (default): every Peerling fights at level 50. It can't be cheated.
- **Real levels:** every Peerling fights at its actual level. Levels aren't
  verified, so a modified client could fake them; both players accept this by
  agreeing to the mode.

In both modes, only verified Peerlings can be used and stat traits apply.

## Consequences
- Levelling up matters in PvP between players who agree to it.
- Cheating in Real-levels mode only affects players who opted in, face to face.
- Each PvP battle records its mode.

## Alternatives considered
- Level 50 only (option A); verified real levels by replaying every battle
  (option B; too heavy); level brackets (option D; needs verified levels).
