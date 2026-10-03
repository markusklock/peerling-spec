---
title: Procedural World Generation
type: system
status: stub
req_prefix: WGN
tags: [world, procedural]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/gameplay/exploration.md
  - wiki/gameplay/encounters.md
  - wiki/peerlings/types.md
updated: 2026-10-03
---

# Procedural World Generation

> How the world the player explores is generated. Status: stub.

[accepted] The world is procedurally generated.

[proposed] Starting assumptions to confirm ([Q-011](../open-questions.md#q-011)):
- Generation is **deterministic from a seed**, so the same seed always produces
  the same world on every client without transferring world data.
- The world is divided into **chunks** generated on demand.
- Each area has a [biome](../glossary.md#biome); biomes map to
  [types](../peerlings/types.md) for encounter weighting.
- Difficulty (wild Peerling levels) increases with distance from the start.

To be specified: seed (shared or per player), world size, biome list,
terrain/props, landmarks, rest/heal points, visual style
([Q-012](../open-questions.md#q-012)).

## Requirements

- **WGN-001** [accepted] The world MUST be procedurally generated.
- **WGN-002** [proposed] World generation MUST be deterministic for a given seed and generator version.

## Open questions

[Q-011](../open-questions.md#q-011) · [Q-012](../open-questions.md#q-012)
