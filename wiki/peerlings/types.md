---
title: Peerling Types
type: reference
status: draft
req_prefix: TYP
tags: [peerlings, battle, balance]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-05-proposal-review-2.md
related:
  - wiki/peerlings/moves.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/gameplay/battle.md
  - wiki/world/procedural-generation.md
updated: 2026-10-05
---

# Peerling Types

> Canonical home of the predefined type list and the type effectiveness chart.
> Types determine battle strengths and weaknesses and which
> [biomes](../glossary.md#biome) a Peerling tends to appear in.

## The type list

[accepted] Types come from a **predefined list**; the LLM picks from it and
cannot invent new types. [accepted] The concept LLM picks a species' type(s) at
the concept stage, based on the player's description
([creation-pipeline § Stage 2](creation-pipeline.md#stage-2--concept)).

[accepted] The list has 12 classic elements. [accepted] Each type has exactly
one home biome, where it appears more often; the biome list is canonical in
[procedural-generation § Biomes](../world/procedural-generation.md#biomes).
Themes are [accepted]:

| Type | Theme |
|------|-------|
| Normal | ordinary animals, everyday things |
| Fire | flame, heat |
| Water | sea, rivers, rain |
| Grass | plants, moss, fungi |
| Electric | lightning, sparks |
| Earth | stone, sand, ground |
| Air | wind, birds, clouds |
| Ice | snow, frost |
| Metal | iron, machines |
| Light | radiance, holiness |
| Shadow | darkness, stealth |
| Spirit | ghosts, dreams, mind |

## Primary and secondary type

[accepted] A dual-typed species lists its types in order: the first is its
**primary** type, the one that best fits the concept. The signature move uses
the primary type ([moves](moves.md#move-slots)).

## Effectiveness chart

[accepted] Approved 2026-10-04.

Rules:
- Multipliers are 2× (super effective), 1× (neutral, shown as ·) and ½× (not
  very effective). There are **no immunities (0×)**, so a player with a single
  Peerling is never completely locked out of a fight.
- Against a dual-typed defender, the two multipliers are multiplied together
  (possible results: 4×, 2×, 1×, ½×, ¼×).
- The multiplier is the "type effectiveness" factor in the
  [damage model](../gameplay/battle.md#damage-model).

Rows are the attacking move's type; columns are the defending Peerling's type.

| Atk ↓ / Def → | Nor | Fir | Wat | Gra | Ele | Ear | Air | Ice | Met | Lig | Sha | Spi |
|---|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| **Normal** | · | · | · | · | · | ½ | · | · | ½ | · | · | ½ |
| **Fire** | · | ½ | ½ | **2** | · | ½ | · | **2** | **2** | · | · | · |
| **Water** | · | **2** | ½ | ½ | · | **2** | · | · | · | · | · | · |
| **Grass** | · | ½ | **2** | ½ | · | **2** | ½ | · | ½ | · | · | · |
| **Electric** | · | · | **2** | ½ | ½ | ½ | **2** | · | · | · | · | · |
| **Earth** | · | **2** | · | ½ | **2** | · | ½ | · | **2** | · | · | ½ |
| **Air** | · | · | · | **2** | ½ | **2** | · | · | · | · | · | · |
| **Ice** | · | ½ | · | **2** | · | · | **2** | ½ | ½ | · | · | · |
| **Metal** | · | ½ | ½ | · | ½ | · | · | **2** | ½ | · | · | **2** |
| **Light** | · | · | · | ½ | · | · | · | **2** | ½ | ½ | **2** | · |
| **Shadow** | **2** | · | · | · | · | · | · | · | · | ½ | ½ | **2** |
| **Spirit** | · | · | · | · | · | · | · | · | ½ | **2** | ½ | **2** |

### Per-type summary

Derived from the chart above (the chart is canonical if they ever differ).

| Type | Strong against (2×) | Weak to (takes 2×) | Resists (takes ½) | Attack: 2× / ½ | Defence: weak / resist |
|---|---|---|---|:-:|:-:|
| Normal | — | Shadow | — | 0 / 3 | 1 / 0 |
| Fire | Grass, Ice, Metal | Water, Earth | Fire, Grass, Ice, Metal | 3 / 3 | 2 / 4 |
| Water | Fire, Earth | Grass, Electric | Fire, Water, Metal | 2 / 2 | 2 / 3 |
| Grass | Water, Earth | Fire, Air, Ice | Water, Grass, Electric, Earth, Light | 2 / 4 | 3 / 5 |
| Electric | Water, Air | Earth | Electric, Air, Metal | 2 / 3 | 1 / 3 |
| Earth | Fire, Electric, Metal | Water, Grass, Air | Normal, Fire, Electric | 3 / 3 | 3 / 3 |
| Air | Grass, Earth | Electric, Ice | Grass, Earth | 2 / 1 | 2 / 2 |
| Ice | Grass, Air | Fire, Metal, Light | Ice | 2 / 3 | 3 / 1 |
| Metal | Ice, Spirit | Fire, Earth | Normal, Grass, Ice, Metal, Light, Spirit | 2 / 4 | 2 / 6 |
| Light | Shadow, Ice | Spirit | Light, Shadow | 2 / 3 | 1 / 2 |
| Shadow | Spirit, Normal | Light | Shadow, Spirit | 2 / 2 | 1 / 2 |
| Spirit | Light, Spirit | Metal, Shadow, Spirit | Normal, Earth | 2 / 2 | 3 / 2 |

### Design notes

- **Familiar core.** Fire, Water, Grass, Electric, Earth, Air and Ice follow
  Pokémon-like intuitions (Water beats Fire, Earth grounds Electric, Ice
  freezes Grass and Air), so most matchups can be guessed without studying.
- **Light → Shadow → Spirit → Light** is a simple cycle: light dispels
  shadow; shadow (nightmares) devours spirits; spirits haunt and dim the
  light. Spirit is also strong against Spirit (ghost against ghost).
- **Metal** is the defensive wall (6 resistances), balanced by weak offence.
  Its Spirit advantage is the folklore of cold iron warding off spirits.
- **Normal** hits nothing super effectively but has only one weakness
  (Shadow: ordinary creatures fear the dark). Since quick moves are often
  Normal type ([moves](moves.md#move-slots)), Normal is the reliable neutral
  option.
- **Ice and Spirit** are the most fragile defensively (three weaknesses and
  only one or two resistances), like Ice in Pokémon.

## Requirements

- **TYP-001** [accepted] There MUST be one predefined list of types (the 12 above); generated species MUST only use types from it.
- **TYP-002** [accepted] Each species MUST have 1 or 2 types; with 2, the order defines primary and secondary.
- **TYP-003** [accepted] Type effectiveness MUST follow the chart above: a multiplier from {2, 1, ½} for every attacker/defender pair, multiplied together against dual-typed defenders.
- ~~**TYP-004**~~ (removed 2026-10-04, replaced by WGN-006 in [procedural-generation](../world/procedural-generation.md#requirements))

## Open questions

_None at the moment._

## See also

- [Moves](moves.md) · [Battle](../gameplay/battle.md)
