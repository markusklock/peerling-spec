---
title: Exploration
type: system
status: draft
req_prefix: EXP
tags: [gameplay, world]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-tech-stack-2.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
  - raw/conversations/2026-10-05-world-details.md
  - raw/conversations/2026-10-05-world-details-approved.md
  - raw/conversations/2026-10-06-peerdex-ui-audio-restpoints.md
related:
  - wiki/world/procedural-generation.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/multiplayer.md
  - wiki/world/visual-style.md
updated: 2026-10-06
---

# Exploration

> How the player moves through and experiences the world: tile-by-tile
> movement on an invisible grid, wild encounters in foliage, and rest points.

[accepted] The player travels around a [procedurally generated world](../world/procedural-generation.md)
looking for Peerlings. [accepted] The world is large but finite. [accepted] The world is shared, so other players
exploring nearby are visible ([multiplayer](multiplayer.md)).

[accepted] The world is seen from a top-down camera in a colorful style
([visual-style](../world/visual-style.md)).

## Grid movement

[accepted] Like Pokémon on the Game Boy, the player moves on an **invisible
grid**, one tile at a time. The grid isn't drawn; the 3D world simply lines up
with it.

[accepted] Details (approved 2026-10-05):
- **Tile size:** 2 m × 2 m. The 4 km world is 2,000 × 2,000 tiles.
- **Directions:** 4 (up, down, left, right), no diagonals, as on the Game Boy.
  Tapping a direction turns the character to face it; holding it walks.
- **Speed:** 3 tiles per second (6 m/s). Each step is a smooth 1/3-second
  animation from tile centre to tile centre; the character is always on exactly
  one tile between steps. Crossing the world takes about 11 minutes.
- **Tile kinds:** each tile is walkable ground, encounter foliage (walkable),
  water, or blocked (trees, rocks, buildings, cliff faces). The world
  generator decides this per tile ([procedural-generation](../world/procedural-generation.md#tiles)).
- **Height:** terrain may rise and fall, but movement stays on the grid.
  Cliffs and ledges sit between tiles, and you can only climb up or down at
  slopes or stairs.
- **Interacting:** the player interacts with the tile they're facing (another
  player, a rest point, the Creation Shrine).
- **Controls:** arrow keys or WASD to move; one key to interact; mouse for
  menus and the battle UI. Full key list: [ui § Controls](ui.md#controls).
- **Other players** share the same grid; two players can stand on the same tile
  (no blocking), so crowds at the spawn never get stuck.

Why: grid movement is simple to control, makes positions tiny to transmit
(tile coordinates, [realtime-networking](../tech/realtime-networking.md#presence-proposed)),
and makes encounter tiles easy to check during verification.

## Wild encounters in foliage

[accepted] Wild Peerlings live in **tall grass, or similar foliage depending on
the biome**. Walking through foliage can trigger an encounter; walking elsewhere
never does. Each biome's foliage is listed in
[procedural-generation § Biomes](../world/procedural-generation.md#biomes).

[accepted] Details (approved 2026-10-05):
- **Trigger chance:** each step onto a foliage tile has a 1 in 10 chance to
  start an encounter. The species and the rest of the encounter come from the
  encounter seed ([encounters](encounters.md)).
- **Grace steps:** after a battle ends, the next 3 steps can't trigger an
  encounter, so the player can't get trapped in back-to-back battles.
- Foliage rustles visibly when the player walks through it, so players learn
  that grass means Peerlings.

## Map

[accepted] A **minimap** and a full **world map** that fill in as the player
explores ([D-0017](../decisions/D-0017-world-features.md)). Discovered rest points and landmarks are marked
by name.

[accepted] The minimap sits in a screen corner; the full map opens with the
**M** key. Exploration is tracked per chunk (32 × 32 tiles): a chunk is
revealed once the player has been in it, and the set of revealed chunks is
stored in the save ([player-data § Save contents](../tech/player-data.md#save-contents)).
The spawn hub and the paths leading out of it are revealed from the start.

## Healing and rest points

[accepted] There are no healing items. A Peerling's HP carries over between
battles and is restored at **rest points**:
- Every biome area has a rest point
  ([procedural-generation § Points of interest](../world/procedural-generation.md#points-of-interest)).
  Visiting it fully heals the whole team, and it becomes the player's *last
  rest point*.
- If the player's whole team faints, the player returns to their last rest
  point with the team fully healed. Nothing is lost.

[accepted] Up close ([D-0019](../decisions/D-0019-peerdex-ui-audio.md)):
- The player faces the beacon and presses interact. The beacon flares, the
  team gets a healing sparkle, the rest-point jingle plays
  ([audio](../world/audio.md#sound-effects)), and a message says: *"Your team is
  fully healed. This is now your rest point."*
- Visiting a rest point also writes a save **snapshot**
  ([player-data § Save log](../tech/player-data.md#save-log)), and now and then
  reminds the player to refresh their backup (file or phone).
- The beacon shows the area's landmark name, biome and level range, and marks
  the landmark on the player's map.
- **No fast travel:** players walk everywhere.

[accepted] The game targets desktop browsers only
([STK-010](../tech/tech-stack.md#requirements)), so controls are designed for
keyboard and mouse.

Points of interest, landmarks, paths and the spawn hub are defined in
[procedural-generation](../world/procedural-generation.md#points-of-interest).

## Requirements

- **EXP-001** [accepted] The player MUST be able to travel freely around the procedurally generated world.
- **EXP-002** [accepted] Visiting a rest point MUST fully heal the player's team and record it as the last rest point.
- **EXP-003** [accepted] When the whole team faints, the player MUST return to the last rest point with the team fully healed, losing nothing.
- **EXP-004** [accepted] The player MUST move on an invisible grid, one tile at a time.
- **EXP-005** [accepted] Wild encounters MUST only be triggered by walking through biome-specific encounter foliage.
- **EXP-006** [accepted] Tiles MUST be 2 m × 2 m; movement MUST be 4-directional at 3 tiles per second; a step onto foliage MUST trigger an encounter with probability 1/10, except during the 3 grace steps after a battle.
- **EXP-007** [accepted] The game MUST have a minimap and a world map that reveal explored areas and mark discovered rest points and landmarks.
- **EXP-008** [accepted] Interacting with a rest point MUST heal the team, set it as the last rest point, and write a save snapshot.
- **EXP-009** [accepted] There MUST NOT be fast travel.

## Open questions

_None at the moment._
