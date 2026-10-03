---
title: Multiplayer — The Shared World
type: system
status: draft
req_prefix: MPL
tags: [gameplay, multiplayer, social]
sources:
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/decisions/D-0008-shared-multiplayer-world.md
  - wiki/gameplay/pvp-battles.md
  - wiki/gameplay/trading.md
  - wiki/tech/realtime-networking.md
  - wiki/world/procedural-generation.md
updated: 2026-10-03
---

# Multiplayer — The Shared World

> All players explore one shared world, see each other, and can battle and
> trade ([D-0008](../decisions/D-0008-shared-multiplayer-world.md)). This page
> covers what players experience; the networking underneath is in
> [realtime-networking](../tech/realtime-networking.md).

## What players experience

- [accepted] Everyone moves around in the **same world**: the same terrain,
  biomes and places for every player.
- [accepted] Players can start a **[PvP battle](pvp-battles.md)** with another
  player.
- [accepted] Players can **[trade](trading.md)** Peerlings with each other.
- [proposed] Nearby players are visible as their
  [player character](player-character.md), with a display name above them.
  Their position and movement update live.
- [proposed] Interacting with another player's character opens a menu:
  *Challenge to battle*, *Propose trade*, *View profile* (their team, and the
  species they created).
- [proposed] Wild encounters are **per player**: each player meets their own
  wild Peerlings, even when standing next to someone else. This avoids
  competing for the same creature.

## Scale and visibility (proposed)

[proposed] Each client shows only players in its own and the neighbouring
[regions](../glossary.md#region), up to a cap. Exact numbers: [Q-027](../open-questions.md#q-027).

## Requirements

- **MPL-001** [accepted] All players MUST share one world.
- **MPL-002** [accepted] Players MUST be able to battle each other and trade Peerlings.
- **MPL-003** [proposed] A client MUST show other players who are near it in the world, with live movement.
- **MPL-004** [proposed] Wild encounters MUST be local to each player; other players MUST NOT be able to interfere with them.
- **MPL-005** [proposed] Battle and trade requests MUST require explicit acceptance by the receiving player, who MUST be able to block or ignore a player.

## Open questions

[Q-025](../open-questions.md#q-025) · [Q-026](../open-questions.md#q-026) ·
[Q-027](../open-questions.md#q-027) · [Q-028](../open-questions.md#q-028)

## See also

- [PvP battles](pvp-battles.md) · [Trading](trading.md) · [Realtime networking](../tech/realtime-networking.md)
