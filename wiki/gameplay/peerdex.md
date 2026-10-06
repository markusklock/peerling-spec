---
title: Peerdex
type: system
status: accepted
req_prefix: DEX
tags: [gameplay, collection, ui]
sources:
  - raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
related:
  - wiki/decisions/D-0019-peerdex-ui-audio.md
  - wiki/gameplay/catching.md
  - wiki/gameplay/sharing.md
  - wiki/gameplay/creator-feedback.md
  - wiki/tech/player-data.md
updated: 2026-10-06
---

# Peerdex

> The player's index of Peerling species: every species they have met, the
> ones they have caught, and their own creations.

[accepted] Decided 2026-10-06 ([D-0019](../decisions/D-0019-peerdex-ui-audio.md)).

## What it lists

The registry keeps growing, so the Peerdex lists only species the player has
**met** (the `seen` events in the save,
[data-formats § Save-log events](../tech/data-formats.md#save-log-events)):

| State | Shown as |
|-------|----------|
| **Seen** (met, not caught) | A silhouette with the name, types, and the biome where it was first met |
| **Caught** | The full card: rotatable 3D model, stats, moves, lore, creator, world stats (met, caught, nodes holding it; [creator-feedback](creator-feedback.md)), CID and a Share button ([sharing](sharing.md)) |

- **Shimmer badge:** shown on species the player has seen or caught as a
  shimmer.
- **Counters:** "Seen 143 · Caught 61 · Species in the world 2,310" (the last is
  the number of active registry entries).
- **Filters and sorting:** by type, biome, creator, newest and name, plus a text
  search.
- **My creations:** a tab with the species the player created and their world
  stats.

## Requirements

- **DEX-001** [accepted] The Peerdex MUST list only species the player has seen, showing seen species as silhouettes (name, types, first biome) and caught species as full cards.
- **DEX-002** [accepted] The Peerdex MUST show seen and caught counts and the number of species in the world, mark shimmer sightings and catches, and offer filters, sorting and search.
- **DEX-003** [accepted] The Peerdex MUST include a "My creations" tab.

## See also

- [Catching](catching.md) · [UI](ui.md)
