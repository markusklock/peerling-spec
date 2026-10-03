---
title: Wild Encounters
type: system
status: draft
req_prefix: ENC
tags: [gameplay, encounters, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-helia.md
  - wiki/world/procedural-generation.md
  - wiki/gameplay/battle.md
updated: 2026-10-03
---

# Wild Encounters

> How the game picks which [wild Peerling](../glossary.md#wild-peerling) the
> player meets, and how its data arrives in time.

## Source of wild Peerlings

[accepted] Wild Peerlings are drawn at random from **all player-created
species**, read from the [registry](../tech/orbitdb-registry.md) and downloaded
via IPFS.

## Selection (proposed)

[proposed] Selection weights to be decided ([Q-017](../open-questions.md#q-017)).
Candidate factors:
- **Biome affinity** — species whose [types](../peerlings/types.md) match the
  current biome are more likely.
- **Novelty** — species the player hasn't seen yet get a boost, so discovery
  stays fresh.
- **Availability** — prefer species whose assets are already cached or
  prefetched, so the encounter starts instantly.
- **Level** — the wild instance's level depends on the region, not the species.

The player's own species MAY appear in the wild (others certainly meet it).

[proposed] Each player meets their own wild Peerlings, even in the shared
world ([MPL-004](multiplayer.md#requirements)).

## Cold start

[accepted] At launch the registry is not empty: the operator creates a handful
of [seed species](../glossary.md#seed-species) through the normal pipeline
([D-0002](../decisions/D-0002-all-peerlings-user-generated.md)). [proposed]
While the registry is small, the same species simply appear repeatedly at
different levels.

## Latency

[proposed] An encounter must never stall on the network. The client keeps a
small **ready pool** of species per nearby biome with assets already fetched
and verified, refilled in the background. If the pool is empty, a cached
species is used.

## Requirements

- **ENC-001** [accepted] Wild Peerlings MUST be chosen from the species in the registry.
- **ENC-002** [accepted] The chosen species' data and model MUST be retrieved via IPFS.
- **ENC-003** [proposed] The client MUST maintain a prefetched pool so an encounter can start without waiting for network retrieval.
- **ENC-004** [proposed] Removed (tombstoned) species MUST NOT be chosen ([REG-005](../tech/orbitdb-registry.md#requirements)).

## Open questions

[Q-017](../open-questions.md#q-017)
