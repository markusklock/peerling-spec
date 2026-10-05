---
title: "D-0014: Individual variation: stat traits and shimmer variants"
type: decision
status: accepted
tags: [peerlings, gameplay, balance, collecting]
sources:
  - raw/conversations/2026-10-05-individual-variation.md
related:
  - wiki/peerlings/peerling-species.md
  - wiki/gameplay/battle.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/trading.md
updated: 2026-10-05
---

# D-0014: Individual variation: stat traits and shimmer variants

**Status:** accepted (2026-10-05, resolves [Q-037](../open-questions.md#q-037));
details [proposed]

## Context
All species have the same base-stat total, and wild Peerlings differ only by
level. So two Peerlings of the same species at the same level were identical,
leaving little reason to hunt for a particular individual or to trade.

## Decision
- [accepted] **Stat traits:** every individual Peerling gets a random
  modifier of up to ±10% on each of its four stats.
- [accepted] **Shimmer variants:** a rare, purely cosmetic color variant.
- [proposed] Rarity 1 in 500; traits are visible to players and count in PvP;
  everything is drawn from verifiable randomness. Details in
  [peerling-species § Individual variation](../peerlings/peerling-species.md#individual-variation).

## Consequences
- Individuals become worth hunting and trading; "fair by construction" now
  applies to species, while individuals differ by up to ±10%.
- Every random part comes from the encounter seed (or from the server for
  starters and shrine creations), so catches stay verifiable by replay.
- Traits and the shimmer belong to the encounter, not to the candidate met
  ([encounters § Wild Peerling generation](../gameplay/encounters.md#wild-peerling-generation)),
  so choosing among the 5 candidates can't be used to chase them. Stalling for
  a new epoch can (at most one re-roll per 5 minutes)
  ([player-data § Known gaps](../tech/player-data.md#known-gaps-accepted-risks)).
- A double trade of a valuable individual hurts the victim more; the cheater is
  still detected and flagged.

## Alternatives considered
- No individual variation (option a), natures (option c): not chosen.
