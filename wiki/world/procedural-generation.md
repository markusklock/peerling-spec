---
title: Procedural World Generation
type: system
status: stub
req_prefix: WGN
tags: [world, procedural]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/gameplay/exploration.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/types.md
updated: 2026-10-04
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

[accepted] The world is **large but finite**: 4 km × 4 km. [proposed] Details:
- Size: [accepted] 4 km × 4 km. At a walking speed of about 5 m/s, crossing it takes
  about 13 minutes, so the world feels big but players still run into each
  other.
- A natural border (ocean, impassable mountains) surrounds it. There are no
  invisible walls.
- One shared spawn area in the centre, where new players meet. Difficulty
  rises toward the edges.
- Being finite makes it possible to grow the world later: a new generator
  version can add an outer ring without changing the existing terrain.

To be specified: biome list, terrain and props, landmarks,
rest/heal points, visual style ([Q-012](../open-questions.md#q-012)), and how a
generator change is rolled out without splitting players into different worlds.

## Requirements

- **WGN-001** [accepted] The world MUST be procedurally generated.
- **WGN-002** [proposed] World generation MUST be deterministic for a given seed and generator version, across browsers and platforms.
- **WGN-003** [accepted] All players MUST be in the same world. [proposed] They MUST therefore use the same global seed.
- **WGN-004** [proposed] Clients with different world-generator versions MUST NOT show each other's presence, so players never see someone walking through terrain that doesn't exist for them.
- **WGN-005** [accepted] The world MUST be finite, 4 km × 4 km. [proposed] It MUST be bounded by a natural border (no invisible walls).

## Open questions

[Q-012](../open-questions.md#q-012)
