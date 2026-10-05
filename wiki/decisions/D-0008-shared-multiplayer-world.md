---
title: "D-0008: One shared world with PvP battles and trading"
type: decision
status: accepted
tags: [gameplay, multiplayer, world, libp2p]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-05-proposal-review-1.md
related:
  - wiki/gameplay/multiplayer.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/tech/realtime-networking.md
  - wiki/world/procedural-generation.md
updated: 2026-10-05
---

# D-0008: One shared world with PvP battles and trading

**Status:** accepted (2026-10-03, resolves [Q-013](../open-questions.md#q-013)
and part of [Q-011](../open-questions.md#q-011))

## Context
The first pass proposed a single-player first version. The designer decided
that all players share one world and can interact.

## Decision
- [accepted] All players move around in the **same procedurally generated
  world**.
- [accepted] Players can **battle each other** and **trade** Peerlings.
- [accepted] This is in scope for the first version.
- [accepted] Player-to-player communication is peer-to-peer over libp2p
  (pubsub for presence, direct streams for battles and trades), with the
  operator server acting only as bootstrap and relay. See
  [realtime-networking](../tech/realtime-networking.md).

## Consequences
- World generation must be deterministic from one global seed, and all clients
  must use the same generator version
  ([procedural-generation](../world/procedural-generation.md)).
- Player saves now affect other players (PvP, trades), so the trust model
  changes ([architecture § Trust model](../tech/architecture.md#trust-model)).
  Cheating and duplication need answers ([Q-025](../open-questions.md#q-025),
  [Q-026](../open-questions.md#q-026)); answered by
  [D-0009](D-0009-player-data-on-orbitdb.md).
- The battle engine must be deterministic so two peers can run the same battle
  and agree on the result ([battle](../gameplay/battle.md)).
- Multiplayer becomes a strong libp2p showcase (seeing players found and
  connected peer-to-peer).

## Alternatives considered
- Single-player world per player: simpler; rejected by the designer.
- Server-authoritative multiplayer (game server): more cheat-resistant, but
  needs more central infrastructure, which goes against pillar 5.
