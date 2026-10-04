---
title: Catching
type: system
status: stub
req_prefix: CAT
tags: [gameplay, catching, collection]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
related:
  - wiki/gameplay/battle.md
  - wiki/peerlings/peerling-species.md
updated: 2026-10-04
---

# Catching

> How a player catches a wild Peerling and what happens to it afterwards.
> Status: stub.

[accepted] Players can catch the Peerlings they encounter.

To be specified: catch chance (HP remaining, status, item used), catching
items and how they are obtained, team size and storage of the rest of the
collection, the collection index ("Peerdex"), and whether creators are credited
or notified ([Q-021](../open-questions.md#q-021)).

[proposed] A caught Peerling becomes a new [instance](../glossary.md#peerling-instance)
in the player's save, referencing its species by CID
([D-0006](../decisions/D-0006-species-vs-instance.md)); its assets are then
retained by the player's node ([NODE-004](../tech/ipfs-helia.md#requirements)).
Caught Peerlings can later be [traded](trading.md).

[accepted] The server verifies each catch afterwards by replaying the battle;
until then the Peerling is *unverified* and can't be traded or used in PvP. See
[player-data § Verification](../tech/player-data.md#verification).

## Requirements

- **CAT-001** [accepted] The player MUST be able to catch wild Peerlings.
- **CAT-002** [accepted] A catch MUST record the catch evidence needed for server verification ([SAVE-006](../tech/player-data.md#requirements)).

## Open questions

[Q-021](../open-questions.md#q-021)
