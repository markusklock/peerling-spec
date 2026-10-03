---
title: Procedural World Generation
type: system
status: stub
req_prefix: WGN
tags: [world, procedural]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/gameplay/exploration.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/types.md
updated: 2026-10-03
---

# Procedural World Generation

> How the world the player explores is generated. Status: stub.

[accepted] The world is procedurally generated. [accepted] There is one shared
world for all players ([D-0008](../decisions/D-0008-shared-multiplayer-world.md)).

[proposed] Starting assumptions to confirm:
- Generation is **deterministic from one global seed**, so every client
  generates the identical world locally without transferring world data. This
  is what makes the shared world possible with no world server.
- The world is divided into **chunks** generated on demand. Chunks are grouped
  into [regions](../glossary.md#region), which scope multiplayer presence
  ([realtime-networking](../tech/realtime-networking.md)).
- Each area has a [biome](../glossary.md#biome); biomes map to
  [types](../peerlings/types.md) for encounter weighting.
- Difficulty (wild Peerling levels) increases with distance from the start.

To be specified: world size (finite or endless,
[Q-011](../open-questions.md#q-011)), biome list, terrain and props, landmarks,
rest/heal points, visual style ([Q-012](../open-questions.md#q-012)), and how a
generator change is rolled out without splitting players into different worlds.

## Requirements

- **WGN-001** [accepted] The world MUST be procedurally generated.
- **WGN-002** [proposed] World generation MUST be deterministic for a given seed and generator version, across browsers and platforms.
- **WGN-003** [accepted] All players MUST be in the same world. [proposed] They MUST therefore use the same global seed.
- **WGN-004** [proposed] Clients with different world-generator versions MUST NOT show each other's presence, so players never see someone walking through terrain that doesn't exist for them.

## Open questions

[Q-011](../open-questions.md#q-011) · [Q-012](../open-questions.md#q-012)
