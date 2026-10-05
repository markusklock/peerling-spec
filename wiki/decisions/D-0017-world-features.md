---
title: "D-0017: World features: spawn hub, landmarks, map, day/night, weather"
type: decision
status: accepted
tags: [world, gameplay, showcase]
sources:
  - raw/conversations/2026-10-05-world-details.md
related:
  - wiki/world/procedural-generation.md
  - wiki/world/visual-style.md
  - wiki/gameplay/exploration.md
updated: 2026-10-05
---

# D-0017: World features: spawn hub, landmarks, map, day/night, weather

**Status:** accepted (2026-10-05); smaller details [proposed]
([Q-046](../open-questions.md#q-046))

## Context
The world had biomes, tiles, foliage and rest points, but no places to go, no
way to find your way, and no plan for updating the world generator.

## Decision
[accepted]
- A **spawn hub** with the Creation Shrine, a rest point, a **New Peerlings
  gallery** and a **network monument**.
- **Rest points** share one beacon silhouette, dressed per biome; every biome
  area also has one big **landmark** with a generated name.
- **Paths**, **bridges** and **signposts** (area name and level range).
- A **minimap** and **world map** that fill in as the player explores.
- **Environment art** is a hand-made low-poly kit shipped with the game app.
- A shared **day/night cycle** and **weather per biome**, driven by the epoch
  clock so every player sees the same.
- **Generator updates** switch at an epoch announced in the epoch records; all
  past generator versions are kept for verification.

Details: [procedural-generation](../world/procedural-generation.md),
[exploration § Map](../gameplay/exploration.md#map),
[visual-style](../world/visual-style.md).

## Alternatives considered
- Environment art made once by the operator with the AI pipeline and stored on
  IPFS: not chosen.
