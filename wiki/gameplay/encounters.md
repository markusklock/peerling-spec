---
title: Wild Encounters
type: system
status: draft
req_prefix: ENC
tags: [gameplay, encounters, ipfs]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
related:
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-helia.md
  - wiki/world/procedural-generation.md
  - wiki/gameplay/battle.md
updated: 2026-10-04
---

# Wild Encounters

> How the game picks which [wild Peerling](../glossary.md#wild-peerling) the
> player meets, and how its data arrives in time.

## Source of wild Peerlings

[accepted] Wild Peerlings are drawn at random from **all player-created
species**, read from the [registry](../tech/orbitdb-registry.md) and downloaded
via IPFS.

## Selection

[accepted] In each [biome](../world/procedural-generation.md#biomes), Peerlings
of that biome's type are more likely to appear.

[proposed] Selection weights (numbers: [Q-017](../open-questions.md#q-017)).
Every eligible species starts with weight 1, then:

| Factor | Multiplier | Why |
|--------|-----------:|-----|
| **Biome affinity:** the species has the biome's type (primary or secondary) | × 6 | If roughly 1 in 12 species has a given type, about a third of a biome's encounters are its type: clearly themed, with plenty of variety |
| **Novelty:** the player has never seen this species (per the Peerdex in the save) | × 2 | Discovery stays fresh as the registry grows |

The species is drawn with these weights using the encounter seed. The wild
level depends on where it is met ([Wild level](#wild-level)), not on the
species.

[proposed] Selection must be a deterministic function of the encounter seed and
data the server can also see, so the server can check it when verifying a catch
([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)). For
that reason "prefer species that are already downloaded" can't be a selection
factor: what a browser has cached is not visible to the server.

The player's own species MAY appear in the wild (others certainly meet it).

[proposed] Each player meets their own wild Peerlings, even in the shared
world ([MPL-004](multiplayer.md#requirements)).

## Wild level

[accepted] Approved 2026-10-04 together with the XP curve. With d = distance in metres from the
world centre (the spawn; the world is 4 km × 4 km, so d is at most about
2,830 m at the corners):

- base level = round(2 + 48 × min(d, 2000) ÷ 2000), i.e. 2 at the spawn and 50
  from 2 km out;
- wild level = base level + a random offset from −2 to +2 (from the encounter
  seed), clamped to 1–50.

## Cold start

[accepted] At launch the registry is not empty: the operator creates a handful
of [seed species](../glossary.md#seed-species) through the normal pipeline
([D-0002](../decisions/D-0002-all-peerlings-user-generated.md)). [proposed]
While the registry is small, the same species simply appear repeatedly at
different levels.

## Latency

[proposed] An encounter must never stall on the network. Because encounters
are determined by the encounter seed, the client can work out in advance which
species its next few encounters would be in the current and neighbouring
biomes, as soon as a new epoch record arrives. It fetches and verifies those species
in the background. If an encounter's species still isn't available when the
encounter triggers, the encounter is delayed (the player keeps walking) until it
is, rather than being replaced by a different species.

## Requirements

- **ENC-001** [accepted] Wild Peerlings MUST be chosen from the species in the registry.
- **ENC-002** [accepted] The chosen species' data and model MUST be retrieved via IPFS.
- **ENC-003** [proposed] The client MUST prefetch the species of its upcoming encounters so an encounter normally starts without waiting for network retrieval.
- **ENC-004** [proposed] Removed (tombstoned) species MUST NOT be chosen ([REG-005](../tech/orbitdb-registry.md#requirements)).
- **ENC-005** [accepted] Encounter species selection and wild level MUST be deterministic functions of the encounter seed, the player's position, the registry state named by the epoch record, and the player's save log.

## Open questions

[Q-017](../open-questions.md#q-017)
