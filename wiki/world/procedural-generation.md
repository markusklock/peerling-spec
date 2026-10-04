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
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
related:
  - wiki/gameplay/exploration.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/types.md
  - wiki/world/visual-style.md
updated: 2026-10-04
---

# Procedural World Generation

> How the world the player explores is generated: one shared, finite world
> built from a global seed, divided into 12 biomes, one per type.

[accepted] The world is procedurally generated. [accepted] There is one shared
world for all players ([D-0008](../decisions/D-0008-shared-multiplayer-world.md)).

## Generation basics

[proposed]
- Generation is **deterministic from one global seed**, so every client
  generates the identical world locally without transferring world data. This
  is what makes the shared world possible with no world server.
- The world is divided into **chunks** generated on demand. Chunks are grouped
  into [regions](../glossary.md#region), which scope multiplayer presence
  ([realtime-networking](../tech/realtime-networking.md)).

## Size and shape

[accepted] The world is **large but finite**: 4 km × 4 km. [proposed] Details:
- At a walking speed of about 5 m/s, crossing it takes about 13 minutes, so the
  world feels big but players still run into each other.
- A natural border (ocean, impassable mountains) surrounds it. There are no
  invisible walls.
- One shared spawn area in the centre, where new players meet. Wild levels
  rise toward the edges ([encounters § Wild level](../gameplay/encounters.md#wild-level)).
- Being finite makes it possible to grow the world later: a new generator
  version can add an outer ring without changing the existing terrain.

## Biomes

[accepted] There are **12 [biomes](../glossary.md#biome), one per
[type](../peerlings/types.md)**. In each biome, Peerlings of its type are more
likely to appear (the weighting is in
[encounters § Selection](../gameplay/encounters.md#selection)).

[accepted] Names and looks (approved 2026-10-04; each biome's dominant colors make it recognizable at
a glance; [visual-style](visual-style.md)):

| Biome | Type | Look |
|-------|------|------|
| Plains | Normal | Rolling green-gold grassland, paths, fences, small farms |
| Volcano | Fire | Black rock, glowing lava streams, red and orange |
| Lakeland | Water | Lakes, rivers, waterfalls and beaches; blues |
| Forest | Grass | Dense woods, mushrooms, mossy logs; deep greens |
| Storm Highlands | Electric | Plateau with crackling crystal spires; yellow and violet |
| Badlands | Earth | Canyons, mesas and sand; ochre and rust |
| Windy Peaks | Air | Cliffs and high ridges with drifting clouds; white and sky blue |
| Tundra | Ice | Snowfields, frozen lakes, glaciers; pale blue |
| Scrapyard Ruins | Metal | Rusted ruins, old machines, gears; steel grey and copper |
| Crystal Meadows | Light | Shining flowers and glowing crystals; white and gold |
| Gloomwood | Shadow | Dark twisted forest in mist; deep purple |
| Haunted Marsh | Spirit | Foggy marsh, will-o'-wisps, old standing stones; teal |

### Layout

[accepted] (approved 2026-10-04)
- The world is split into biome **areas** about 300–500 m across (e.g. Voronoi
  cells around points scattered by the seed), giving roughly 80–150 areas.
- **Every biome appears at every distance from the centre.** The world is
  divided into rings (0–500 m, 500–1000 m, 1000–1500 m, 1500–2000 m, beyond
  2000 m), and each ring contains areas of all 12 biomes. Since wild levels
  depend on distance, this means every type can be found at every level range,
  not just Ice Peerlings far out.
- The spawn area (about 150 m around the centre) is Plains.
- Borders between areas blend over a short distance, so biomes flow into each
  other rather than switching abruptly.
- Every biome area contains one **rest point**
  ([exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points)).

To be specified: terrain shapes and props per biome, landmarks, and how a generator change is
rolled out without splitting players into different worlds.

## Requirements

- **WGN-001** [accepted] The world MUST be procedurally generated.
- **WGN-002** [proposed] World generation MUST be deterministic for a given seed and generator version, across browsers and platforms.
- **WGN-003** [accepted] All players MUST be in the same world. [proposed] They MUST therefore use the same global seed.
- **WGN-004** [proposed] Clients with different world-generator versions MUST NOT show each other's presence, so players never see someone walking through terrain that doesn't exist for them.
- **WGN-005** [accepted] The world MUST be finite, 4 km × 4 km. [proposed] It MUST be bounded by a natural border (no invisible walls).
- **WGN-006** [accepted] There MUST be 12 biomes, one for each type, and each biome MUST raise the chance of encountering Peerlings of its type.
- **WGN-007** [accepted] Every biome MUST occur in every distance ring, so every type can be met at every level range.
- **WGN-008** [accepted] The spawn area MUST be Plains.
- **WGN-009** [accepted] Every biome area MUST contain one rest point.

## Open questions

_None at the moment._
