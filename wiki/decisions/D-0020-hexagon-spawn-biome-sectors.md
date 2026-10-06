---
title: "D-0020: Central spawn hexagon with 12 biome sectors"
type: decision
status: accepted
tags: [world, generation, biomes]
sources:
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-pvp-wins-hex-world.md
related:
  - wiki/world/procedural-generation.md
  - wiki/gameplay/encounters.md
updated: 2026-10-06
---

# D-0020: Central spawn hexagon with 12 biome sectors

**Status:** accepted (2026-10-06). Replaces the ring-based biome layout of
2026-10-04 ([procedural-generation § Layout](../world/procedural-generation.md#layout)).

## Context
The earlier layout split the world into distance rings, each containing areas
of all 12 biomes, so every type could be met at every level range (wild levels
depend on the distance from the centre). A consistency review found that the
inner ring (0–500 m) is too small to hold 12 areas of 300–500 m. The designer
suggested building the world from a central spawn hexagon, where Normal
Peerlings are more common, with the 12 biomes around it.

## Decision
[accepted] Option B:
- A **central Plains hexagon** where everyone spawns; the spawn hub sits in
  its middle.
- Around it, **12 wedge-shaped biome sectors**, one per biome (Plains
  included), each reaching from the hexagon to the world's border.

Exact size, orientation and biome order: [proposed], see
[procedural-generation § Layout](../world/procedural-generation.md#layout).

## Consequences
- Every type is still found at every level range outside the hexagon, because
  every sector runs from the centre to the border.
- The world map reads as a compass: the direction from the centre picks the
  biome, the distance picks the level.
- New players start among Plains (Normal-heavy) Peerlings at low levels.
- WGN-007 (biomes in every distance ring) is replaced by WGN-017.

## Alternatives considered
- **A. 13 hexagons literally** (Plains in the centre, the others in rings
  around it): each biome would sit at only one distance, so its type would be
  common at only one level range.
- **C. A hexagonal grid of small biome areas** with Plains in the centre:
  keeps every biome at every distance, but loses the clear 12 + 1 shape.
- **Keep the ring layout** with smaller inner areas: doesn't fix the
  crowded inner ring cleanly.
