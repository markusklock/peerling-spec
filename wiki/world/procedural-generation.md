---
title: Procedural World Generation
type: system
status: draft
req_prefix: WGN
tags: [world, procedural]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
  - raw/conversations/2026-10-05-proposal-review-1.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-05-world-details.md
  - raw/conversations/2026-10-05-world-details-approved.md
  - raw/conversations/2026-10-06-review-decisions.md
related:
  - wiki/gameplay/exploration.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/multiplayer.md
  - wiki/peerlings/types.md
  - wiki/world/visual-style.md
updated: 2026-10-06
---

# Procedural World Generation

> How the world the player explores is generated: one shared, finite world
> built from a global seed, divided into 12 biomes, one per type, with a spawn
> hub, landmarks, paths, a shared day/night cycle and weather.

[accepted] The world is procedurally generated. [accepted] There is one shared
world for all players ([D-0008](../decisions/D-0008-shared-multiplayer-world.md)).

## Generation basics

[accepted]
- Generation is **deterministic from one global seed**, so every client
  generates the identical world locally without transferring world data. This
  is what makes the shared world possible with no world server.
- The world is divided into **chunks** generated on demand. [accepted] A chunk
  is 32 × 32 tiles (64 m × 64 m), and each chunk is also a
  [region](../glossary.md#region), the unit that scopes multiplayer presence
  ([realtime-networking](../tech/realtime-networking.md)).

## Tiles

[accepted] The world is laid out on an invisible grid of tiles that players
move across one at a time ([exploration § Grid movement](../gameplay/exploration.md#grid-movement)).

[accepted] Each 2 m × 2 m tile has one kind, decided by the generator:

| Kind | Walkable | Notes |
|------|:--------:|-------|
| Ground | Yes | Paths, open ground, sand, snow |
| Foliage | Yes | The biome's encounter foliage (below); wild encounters happen only here |
| Water | No | Lakes, rivers, sea at the border (no swimming in v1) |
| Blocked | No | Trees, rocks, buildings, cliff faces, lava |

[accepted] Each tile also has a height level; neighbouring tiles at different
heights are separated by a cliff unless one of them is a slope or stairs.
About 20–30% of walkable tiles in a biome area are foliage, in patches of 10–60
tiles, with paths of plain ground around and between them. Rest points and the
Creation Shrine are never surrounded by foliage.

## Size and shape

[accepted] The world is **large but finite**: 4 km × 4 km. [accepted] Details:
- At the walking speed of 3 tiles per second (6 m/s,
  [exploration § Grid movement](../gameplay/exploration.md#grid-movement)),
  crossing it takes about 11 minutes, so the world feels big but players still
  run into each other.
- A natural border (ocean, impassable mountains) surrounds it. There are no
  invisible walls.
- One shared spawn area in the centre, where new players meet. Wild levels
  rise toward the edges ([encounters § Wild level](../gameplay/encounters.md#wild-level)).
- [accepted] The world does not grow in v1: its size is fixed, because wild
  levels and the biome layout depend on it.

## Biomes

[accepted] There are **12 [biomes](../glossary.md#biome), one per
[type](../peerlings/types.md)**. In each biome, Peerlings of its type are more
likely to appear (the weighting is in
[encounters § Selection](../gameplay/encounters.md#selection)).

[accepted] Names and looks (approved 2026-10-04; each biome's dominant colors make it recognizable at
a glance; [visual-style](visual-style.md)):

| Biome | Type | Look | Encounter foliage [accepted] |
|-------|------|------|------------------|
| Plains | Normal | Rolling green-gold grassland, paths, fences, small farms | Tall grass |
| Volcano | Fire | Black rock, glowing lava streams, red and orange | Ember-tipped ash grass |
| Lakeland | Water | Lakes, rivers, waterfalls and beaches; blues | Reeds along the shores |
| Forest | Grass | Dense woods, mushrooms, mossy logs; deep greens | Ferns and undergrowth |
| Storm Highlands | Electric | Plateau with crackling crystal spires; yellow and violet | Crackling static grass |
| Badlands | Earth | Canyons, mesas and sand; ochre and rust | Dry scrub |
| Windy Peaks | Air | Cliffs and high ridges with drifting clouds; white and sky blue | Wind-swept tall grass |
| Tundra | Ice | Snowfields, frozen lakes, glaciers; pale blue | Snow-covered shrubs |
| Scrapyard Ruins | Metal | Rusted ruins, old machines, gears; steel grey and copper | Overgrown scrap heaps |
| Crystal Meadows | Light | Shining flowers and glowing crystals; white and gold | Glowing flower beds |
| Gloomwood | Shadow | Dark twisted forest in mist; deep purple | Dark brambles |
| Haunted Marsh | Spirit | Foggy marsh, will-o'-wisps, old standing stones; teal | Misty marsh grass |

### Layout

[accepted] (approved 2026-10-04)
- The world is split into biome **areas** about 300–500 m across (e.g. Voronoi
  cells around points scattered by the seed), giving roughly 80–150 areas.
- **Every biome appears at every distance from the centre.** The world is
  divided into rings (0–500 m, 500–1000 m, 1000–1500 m, 1500–2000 m, beyond
  2000 m), and each ring contains areas of all 12 biomes. Since wild levels
  depend on distance, this means every type can be found at every level range,
  not just Ice Peerlings far out. The inner ring is too small for areas of all
  12 biomes; a new layout is under discussion
  ([Q-053](../open-questions.md#q-053)).
- The spawn area (about 150 m around the centre) is Plains.
- Borders between areas blend over a short distance, so biomes flow into each
  other rather than switching abruptly.
- Every biome area contains one **rest point**
  ([exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points)).

## Terrain and props

[accepted] ([D-0017](../decisions/D-0017-world-features.md))
- **Height** comes from noise with a different character per biome: gentle
  rolling ground in Plains and Crystal Meadows, mesas and canyons in the
  Badlands, steep cliffs and ridges in the Windy Peaks, low wetlands in
  Lakeland and the Haunted Marsh, a cone with lava channels in Volcano areas.
  Heights are whole levels per tile, with cliffs between tiles
  ([Tiles](#tiles)).
- **Border:** the world is surrounded by an ocean ring, with mountains in
  places.
- **Props** (trees, rocks, ruins, crystals, scrap) come from a per-biome kit
  and are placed deterministically from the world seed. The art is a
  hand-made low-poly kit shipped with the game app
  ([visual-style](visual-style.md#environment-art)).

## Spawn hub

[accepted] The centre of the world, where everyone starts, is the game's social
and showcase heart ([D-0017](../decisions/D-0017-world-features.md)):
- **Creation Shrine:** a glowing stone circle ([creation-shrine](../gameplay/creation-shrine.md)).
- **Rest point.**
- **New Peerlings gallery:** pedestals showing the most recently published
  Peerlings as 3D models, loaded live from IPFS and updated as new ones appear.
  Clicking one opens its card ([sharing](../gameplay/sharing.md)).
  [accepted] 12 pedestals, newest first.
- **Network monument:** a crystal tree whose glowing branches show live network
  activity. [accepted] Each glowing branch is a peer the player is connected
  to; the number of leaves follows how many Peerlings this computer is storing
  and serving; it pulses when content is served to someone.

[accepted] The hub is about 30 × 30 tiles of paved ground with no foliage, so
nobody meets wild Peerlings in the crowd.

## Points of interest

[accepted] ([D-0017](../decisions/D-0017-world-features.md))
- **Rest points:** one per biome area ([WGN-009](#requirements)), all with the
  same recognisable silhouette, a glowing lantern beacon, dressed for their
  biome (snow-covered in the Tundra, vine-wrapped in the Forest, rusted in
  Scrapyard Ruins…). They heal the team and set the respawn point
  ([exploration § Healing and rest points](../gameplay/exploration.md#healing-and-rest-points)).
- **Landmarks:** one large, unique landmark per biome area (a crater, giant
  waterfall, ancient crystal, wrecked machine, haunted tower…), built from the
  biome's kit and placed by the seed. Each has a **generated name** (e.g.
  "Ember Crater", "Whispering Falls") so players can say where to meet.
  [accepted] Names are built deterministically from per-biome word lists (an
  adjective-like part and a feature noun, e.g. *Whispering* + *Falls*), and are
  unique within the world.
- **Paths and bridges:** paths connect the rest points and the spawn hub, with
  bridges where they cross rivers. They mostly avoid foliage, so players can
  travel without constant encounters. [accepted] The path network links each
  rest point to its nearest neighbours (a minimum spanning tree plus a few
  extra links so there are loops), and paths may cross foliage only where no
  other route exists.
- **Signposts:** at path junctions and area borders, showing the area's landmark
  name, biome and level range (e.g. *"Gloomwood: wild Peerlings level 22–26"*).
  [accepted] The range is the lowest base level in the area minus 2 to the
  highest base level plus 2 (the random offset), clamped to 1–50
  ([encounters § Wild level](../gameplay/encounters.md#wild-level)).

## Day and night

[accepted] A shared day/night cycle driven by the epoch clock, so every player
sees the same time of day ([D-0017](../decisions/D-0017-world-features.md)).

[accepted] One in-game day lasts 24 epochs (2 hours). The time of day is
computed from Unix time, so it is smooth and identical everywhere without any
messages. Night darkens the scene and lights up rest points, landmarks and
glowing foliage. It is purely visual in v1 (no effect on encounters).

[proposed] Exact clock, awaiting approval ([Q-051](../open-questions.md#q-051)):
day phase = Unix time in ms mod 7,200,000. Phase 0 is midnight and 3,600,000 is
noon; each in-game hour is 300,000 ms (one epoch). Night runs from 18:00 to
06:00 in-game time (phase below 1,800,000 or from 5,400,000).

## Weather

[accepted] Weather per biome, the same for every player ([D-0017](../decisions/D-0017-world-features.md)).

[accepted] Each biome's weather changes every 3 epochs (15 minutes). The next
state is chosen from the epoch record's randomness (drand) and the biome, so
all clients agree, including while the server is offline (client-derived epoch
records, [player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)).
Purely visual in v1.

[proposed] Exact selection, awaiting approval ([Q-051](../open-questions.md#q-051)):
- Weather periods start at every epoch number divisible by 3. The period uses
  the `randomness` of the epoch record of its first epoch.
- h = SHA-256(`"peerlings/weather/v1"` ‖ that randomness ‖ the biome's index
  as one byte), where the index is the biome's row (0–11) in the table below.
- Weights: the first state listed for the biome has weight 3, each other
  state 1. Aurora has weight 0 unless the period starts at night (see
  [Day and night](#day-and-night)).
- The state = the first 4 bytes of h as a big-endian unsigned integer, mod
  the total weight, mapped onto the states in table order.

| Biome | Weather states [accepted] |
|-------|----------------|
| Plains | Clear, cloudy, light rain |
| Volcano | Clear, ash fall, ember storm |
| Lakeland | Clear, rain, morning fog |
| Forest | Clear, rain, mist |
| Storm Highlands | Clear, thunderstorm |
| Badlands | Clear, heat haze, sandstorm |
| Windy Peaks | Clear, strong wind, low clouds |
| Tundra | Clear, snowfall, blizzard |
| Scrapyard Ruins | Overcast, rust-colored rain |
| Crystal Meadows | Clear, sparkle shower, aurora (night only) |
| Gloomwood | Mist, thick fog |
| Haunted Marsh | Fog, will-o'-wisp swarms |

## Generator updates

[accepted] How the world generator is updated without splitting players into
different worlds ([D-0017](../decisions/D-0017-world-features.md)):
- **Versions.** Each world-generator version has a version number. Players on
  different versions don't see each other (WGN-004).
- **A shared switch time.** An update is announced in the epoch records as
  "from epoch X, the world uses generator version N"
  ([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)).
  Every client switches at that epoch; a client without the new version asks
  the player to reload, and the installed app updates itself.
- **Old versions are kept.** Verifying a catch needs the world as it was at
  the catch's epoch (was that tile foliage?), so the game app contains every
  past generator version, and verifiers use the one that was active.
- **Nobody gets stuck.** A player standing on a tile that became blocked is
  moved to the nearest walkable tile. A last rest point that no longer exists
  becomes the nearest one.
- **Prefer additive changes:** new landmarks and details rather than
  reshaping existing land, so players' knowledge of the world stays valid.

## Requirements

- **WGN-001** [accepted] The world MUST be procedurally generated.
- **WGN-002** [accepted] World generation MUST be deterministic for a given seed and generator version, across browsers and platforms.
- **WGN-003** [accepted] All players MUST be in the same world. [accepted] They MUST therefore use the same global seed.
- **WGN-004** [accepted] Clients with different world-generator versions MUST NOT show each other's presence, so players never see someone walking through terrain that doesn't exist for them.
- **WGN-005** [accepted] The world MUST be finite, 4 km × 4 km. [accepted] It MUST be bounded by a natural border (no invisible walls).
- **WGN-006** [accepted] There MUST be 12 biomes, one for each type, and each biome MUST raise the chance of encountering Peerlings of its type.
- **WGN-007** [accepted] Every biome MUST occur in every distance ring, so every type can be met at every level range.
- **WGN-008** [accepted] The spawn area MUST be Plains.
- **WGN-009** [accepted] Every biome area MUST contain one rest point.
- **WGN-010** [accepted] The world MUST be a grid of tiles, and each biome MUST have its own encounter foliage on which wild encounters happen.
- **WGN-011** [accepted] Tiles MUST be 2 m × 2 m and of one kind (ground, foliage, water, blocked) with a height level; a chunk MUST be 32 × 32 tiles.
- **WGN-012** [accepted] The world MUST have a spawn hub with the Creation Shrine, a rest point, a New Peerlings gallery loaded live from IPFS, and a network monument showing live network activity.
- **WGN-013** [accepted] Every biome area MUST have one landmark with a generated name; rest points MUST share one silhouette dressed per biome.
- **WGN-014** [accepted] Paths MUST connect rest points and the spawn hub, with bridges over water and signposts showing area name, biome and level range.
- **WGN-015** [accepted] The day/night cycle and per-biome weather MUST be the same for all players, derived from time and the epoch records. [accepted] A day lasts 24 epochs; weather changes every 3 epochs; both are purely visual in v1.
- **WGN-016** [accepted] Generator updates MUST switch at an epoch announced in the epoch records, and clients MUST keep all past generator versions for verification.

## Open questions

- [Q-051](../open-questions.md#q-051) — exact day clock and weather selection
- [Q-053](../open-questions.md#q-053) — world layout: central spawn hexagon with 12 biomes around it?
