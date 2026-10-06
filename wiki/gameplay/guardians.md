---
title: Landmark Guardians and Badges
type: system
status: draft
req_prefix: GRD
tags: [gameplay, battle, progression, world]
sources:
  - raw/conversations/2026-10-06-v1-fun-features.md
related:
  - wiki/decisions/D-0022-v1-fun-features.md
  - wiki/gameplay/battle.md
  - wiki/world/procedural-generation.md
  - wiki/tech/player-data.md
updated: 2026-10-06
---

# Landmark Guardians and Badges

> One guardian per biome, whose team of that biome's Peerlings changes every
> week. Beating a guardian earns the biome's badge; there are 12 badges.

[accepted] Each biome has a guardian that players battle for that biome's
**badge**, so there are 12 badges in all. The guardian's team is drawn each
week from that biome's Peerlings using the shared epoch randomness. Badge wins
are verified by replay, like catches, and no server is needed
([D-0022](../decisions/D-0022-v1-fun-features.md)).

Why: after catching and levelling, v1 had no goals to complete. Badges give a
clear progression goal, like gyms in Pokémon. The weekly team gives a reason
to come back, and it shows off player creations: every week, 48 species stand
guard somewhere in the world.

## Guardian sites

[proposed] Details below await approval ([Q-055](../open-questions.md#q-055)).

- **One site per [biome sector](../glossary.md#biome-sector)**
  ([procedural-generation § Layout](../world/procedural-generation.md#layout)).
  The site replaces the landmark of the biome area it is in, so it gets a
  generated name like other landmarks (e.g. *"Ember Crater Guardian Stones"*).
- **Look:** a raised stone platform dressed for the biome, with a guardian
  statue at its head and four plinths showing this week's team as 3D models,
  loaded live from IPFS. The statue glows if the player already holds the
  badge.
- **Placement:** sector *i* (in the clockwise sector order, Plains = 0) has
  its site in the biome area nearest to the point at distance
  d = 350 + 150 × i metres from the world centre, on the sector's middle line.
  Going round the compass clockwise, the guardians get stronger, ending at
  the world's edge:

  | Sector *i* | Biome | Distance | Base level there |
  |-----:|-------|-------:|------:|
  | 0 | Plains | 350 m | 10 |
  | 1 | Forest | 500 m | 14 |
  | 2 | Gloomwood | 650 m | 18 |
  | 3 | Haunted Marsh | 800 m | 21 |
  | 4 | Lakeland | 950 m | 25 |
  | 5 | Tundra | 1,100 m | 28 |
  | 6 | Windy Peaks | 1,250 m | 32 |
  | 7 | Storm Highlands | 1,400 m | 36 |
  | 8 | Crystal Meadows | 1,550 m | 39 |
  | 9 | Scrapyard Ruins | 1,700 m | 43 |
  | 10 | Badlands | 1,850 m | 46 |
  | 11 | Volcano | 2,000 m | 50 |

  The badges can be earned in any order; the table is only the natural route.
- The site's biome area keeps its rest point, so a defeated player respawns
  nearby.

## The weekly team

[proposed]
- **Week** W = floor(E ÷ 2016), where E is the epoch number (2016 epochs =
  7 days). The team for week W is defined by the
  [epoch record](../tech/data-formats.md#epoch-record--peerlingsepoch) of
  epoch 2016 × W (signed or client-derived, as for encounters).
- **Species:** the eligible species at that record's `registryHeight`
  ([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)) that
  have the biome's type (primary or secondary), sorted by registry `seq`. Four
  different species are drawn uniformly without replacement with the
  [random number generator](battle.md#random-number-generator), seeded with
  SHA-256(`"peerlings/guardian/v1"` ‖ the record's randomness ‖ the
  [biome index](../world/procedural-generation.md#biomes) as one byte). If
  fewer than 4 have the type, the rest are drawn the same way from all other
  eligible species.
- **Individuals:** every member is at the guardian level, with all traits 0
  and no shimmer. Team order is the draw order.
- **Guardian level** = min(50, the base level at the site's statue tile + 3)
  ([encounters § Wild level](encounters.md#wild-level)).
- **Creators:** a creator whose species is drawn sees *"Your Mossnap guards
  the Forest this week!"* (worked out by their own client).

## The battle

[proposed]
- The player faces the statue and presses interact. A preview shows the
  guardian level and the four species (names, types); the player can then
  accept the challenge.
- It follows the [battle rules](battle.md#rules) for wild battles, except:
  - the guardian side has 4 Peerlings, sent out in team order; when one
    faints, the next comes out without using a turn;
  - each guardian Peerling chooses moves like a
    [wild Peerling](battle.md#wild-peerling-behaviour) and never switches;
  - the player can't catch; fleeing ends the challenge with no penalty.
- **XP:** each guardian Peerling defeated gives XP as a wild Peerling of its
  level ([battle § Experience and levelling](battle.md#experience-and-levelling)).
- **Losing** works like losing a wild battle: back to the last rest point,
  fully healed. HP carries over afterwards, as after any wild battle.
- **Randomness:** the battle's rolls use the
  [encounter seed](../tech/player-data.md#encounter-seeds) of the player's
  next encounter number, so a guardian battle can't be re-rolled and is
  logged like an encounter. There are no species, level or trait draws; the
  battle rolls start straight away.
- **Winning** earns the badge if the player doesn't have it yet. Guardians
  can be fought again at any time, for XP and for fun.

## Badges

[proposed]
- A won guardian battle is logged as a `battle-result` with the biome; the
  first win per biome also writes a `badge` event with the evidence needed to
  replay it ([data-formats § Save-log events](../tech/data-formats.md#save-log-events)).
- A badge counts only if the replay verifies, like a catch
  ([player-data § Verification](../tech/player-data.md#verification)).
- Badges appear in a badge case on the player's profile, in game and in the
  shared [profile document](../tech/data-formats.md#player-profile-document--peerlingsprofile).
- Earning all 12 badges posts a world feed event
  ([world-feed](world-feed.md)).

## Requirements

- **GRD-001** [accepted] Each biome MUST have one guardian whose team is drawn each week from that biome's Peerlings using the shared epoch randomness, without a server.
- **GRD-002** [accepted] Beating a guardian MUST earn that biome's badge (12 badges in all), and a badge MUST only count if its battle verifies by replay.
- **GRD-003** [proposed] Guardian sites, weekly teams, the battle and badges MUST follow the rules on this page.

## Open questions

- [Q-055](../open-questions.md#q-055) — approve the exact rules for the v1 fun features

## See also

- [Battle](battle.md) · [Peerling of the Day](peerling-of-the-day.md) · [Procedural generation](../world/procedural-generation.md)
