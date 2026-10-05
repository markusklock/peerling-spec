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
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-04-answers-round-8.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
related:
  - wiki/tech/orbitdb-registry.md
  - wiki/tech/ipfs-helia.md
  - wiki/world/procedural-generation.md
  - wiki/gameplay/battle.md
  - wiki/tech/resilience.md
updated: 2026-10-05
---

# Wild Encounters

> How the game picks which [wild Peerling](../glossary.md#wild-peerling) the
> player meets, and how its data arrives in time.

## Source of wild Peerlings

[accepted] Wild Peerlings are drawn at random from **all player-created
species**, read from the [registry](../tech/orbitdb-registry.md) and downloaded
via IPFS.

[accepted] Encounters happen only in the biome's encounter foliage (tall grass
or similar). What triggers one is defined in
[exploration § Wild encounters in foliage](exploration.md#wild-encounters-in-foliage).
The biome used for [selection](#selection) is the biome of the foliage tile
where the encounter started, and the distance for the [wild level](#wild-level)
is measured from that tile's centre.

## Selection

[accepted] In each [biome](../world/procedural-generation.md#biomes), Peerlings
of that biome's type are more likely to appear.

[accepted] Selection weights (approved 2026-10-04).
Every eligible species starts with weight 1, then:

| Factor | Multiplier | Why |
|--------|-----------:|-----|
| **Biome affinity:** the species has the biome's type (primary or secondary) | × 6 | If roughly 1 in 12 species has a given type, about a third of a biome's encounters are its type: clearly themed, with plenty of variety |
| **Novelty:** the player has never seen this species (per the Peerdex in the save) | × 2 | Discovery stays fresh as the registry grows |

The candidates (below) are drawn with these weights using the encounter
seed. The wild level depends on where it is met ([Wild level](#wild-level)),
not on the species.

[accepted] Selection must be a deterministic function of the encounter seed and
data the server can also see, so the server can check it when verifying a catch
([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)).

## Candidates

[accepted] Each encounter has **5 candidate** Peerlings rather than one. The
game tries to fetch them via IPFS and the encounter uses one that arrives. This
makes encounters faster and keeps them working when some species can't be
downloaded, e.g. while the operator server (the only node pinning everything)
is offline ([resilience](../tech/resilience.md)).

[accepted] To keep catches verifiable, the candidates and the choice between
them follow fixed rules:
1. **An ordered list.** The encounter seed draws 5 different species, one after
   another, using the selection weights (weighted draws without replacement).
   If fewer than 5 species are eligible, all of them are candidates.
2. **Prefetching.** The client works out its upcoming encounters' candidate
   lists in advance and fetches them, in list order, in the background.
3. **Choice.** When the encounter triggers, it uses the candidate **earliest
   in the list** that has already been fetched and verified. If none has arrived
   yet, it uses the first one that does arrive, as in the designer's original
   idea.
4. **Nothing arrives.** If no candidate arrives within 10 seconds, no encounter
   happens and the encounter number isn't used up. The next encounter trigger
   tries the same 5 candidates again.
5. **Logging.** The save log records which candidate (1–5) was met; the server
   checks it is on the list.

Why "earliest in the list" instead of always "first to download": with
prefetching, several candidates are usually already downloaded, so "first to
download" is unclear. Earliest in the list keeps the outcome stable and the
server can check it. A modified client could still pick any of the 5; that
known gap is listed in
[player-data § Known gaps](../tech/player-data.md#known-gaps-accepted-risks).

The player's own species MAY appear in the wild (others certainly meet it).

[accepted] Each player meets their own wild Peerlings, even in the shared
world ([MPL-004](multiplayer.md#requirements)).

## Wild level

[accepted] Approved 2026-10-04 together with the XP curve. With d = distance in metres from the
world centre (the spawn; the world is 4 km × 4 km, so d is at most about
2,830 m at the corners):

- base level = round(2 + 48 × min(d, 2000) ÷ 2000), i.e. 2 at the spawn and 50
  from 2 km out;
- wild level = base level + a random offset from −2 to +2 (from the encounter
  seed), clamped to 1–50.

## Wild Peerling generation

[accepted] Once the species is chosen, the wild Peerling is generated from the
same encounter seed and [random number generator](battle.md#random-number-generator),
always in this order, so a replay produces exactly the same individual:

1. The 5 candidate species ([Candidates](#candidates)).
2. The level offset (−2 to +2) ([Wild level](#wild-level)).
3. The four stat traits, in the order HP, Attack, Defense, Speed
   ([peerling-species § Individual variation](../peerlings/peerling-species.md#individual-variation)).
4. The shimmer roll (1 in 500).
5. Then all battle rolls.

The candidate actually met only changes the species: the level, traits and
shimmer belong to the encounter, whichever candidate is met.

## Cold start

[accepted] At launch the registry is not empty: the operator creates a handful
of [seed species](../glossary.md#seed-species) through the normal pipeline
([D-0002](../decisions/D-0002-all-peerlings-user-generated.md)). [accepted]
While the registry is small, the same species simply appear repeatedly at
different levels.

## Latency

[accepted] An encounter must never stall on the network. As soon as a new
epoch record arrives, the client works out the candidate lists of its next few
encounters in the current and neighbouring biomes and fetches them in the
background ([Candidates](#candidates)). With 5 candidates prefetched, at least
one is almost always ready when an encounter triggers.

## Requirements

- **ENC-001** [accepted] Wild Peerlings MUST be chosen from the species in the registry.
- **ENC-002** [accepted] The chosen species' data and model MUST be retrieved via IPFS.
- **ENC-003** [accepted] The client MUST prefetch the species of its upcoming encounters so an encounter normally starts without waiting for network retrieval.
- **ENC-004** [accepted] Removed (tombstoned) species MUST NOT be chosen ([REG-005](../tech/orbitdb-registry.md#requirements)).
- **ENC-005** [accepted] Encounter species selection and wild level MUST be deterministic functions of the encounter seed, the player's position, the registry state named by the epoch record, and the player's save log.
- **ENC-006** [accepted] Each encounter MUST have up to 5 candidate species, and the encounter MUST use a candidate that was successfully fetched via IPFS.
- **ENC-007** [accepted] Candidates MUST form an ordered list drawn from the encounter seed; the encounter MUST use the earliest candidate already fetched, or else the first to arrive; if none arrives within 10 s, no encounter happens and the encounter number is not consumed.
- **ENC-008** [accepted] Wild Peerlings MUST be generated from the encounter seed in the order given in [Wild Peerling generation](#wild-peerling-generation).

## Open questions

_None at the moment._
