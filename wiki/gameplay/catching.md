---
title: Catching
type: system
status: stub
req_prefix: CAT
tags: [gameplay, catching, collection]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/gameplay/battle.md
  - wiki/peerlings/peerling-species.md
updated: 2026-10-03
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

## Requirements

- **CAT-001** [accepted] The player MUST be able to catch wild Peerlings.

## Open questions

[Q-010](../open-questions.md#q-010) · [Q-021](../open-questions.md#q-021)
