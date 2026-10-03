---
title: Peerling Types
type: reference
status: draft
req_prefix: TYP
tags: [peerlings, battle, balance]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
related:
  - wiki/peerlings/moves.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/gameplay/battle.md
  - wiki/world/procedural-generation.md
updated: 2026-10-03
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

[accepted] The list has 12 classic elements. The themes and biomes are still
[proposed]:

| Type | Theme | Typical biomes [proposed] |
|------|-------|----------------|
| Normal | ordinary animals, everyday things | plains, towns |
| Fire | flame, heat | volcanic, desert |
| Water | sea, rivers, rain | coast, lakes |
| Grass | plants, moss, fungi | forest, meadow |
| Electric | lightning, sparks | plains, storm peaks |
| Earth | stone, sand, ground | mountains, canyons |
| Air | wind, birds, clouds | cliffs, high plateaus |
| Ice | snow, frost | tundra, glaciers |
| Metal | iron, machines | ruins, caves |
| Light | radiance, holiness | open skies, temples |
| Shadow | darkness, stealth | caves, deep forest |
| Spirit | ghosts, dreams, mind | ruins, night |

## Primary and secondary type

[proposed] A dual-typed species lists its types in order: the first is its
**primary** type, the one that best fits the concept. The signature move uses
the primary type ([moves](moves.md#move-slots)).

## Effectiveness chart

TBD ([Q-008](../open-questions.md#q-008)). [proposed] Structure: attacker type ×
defender type → multiplier from {2×, 1×, 0.5×}. There are no full immunities
(0×), so a player with a single creature is never locked out of a fight. For
dual-typed defenders, the two multipliers are multiplied together.

## Requirements

- **TYP-001** [accepted] There MUST be one predefined list of types (the 12 above); generated species MUST only use types from it.
- **TYP-002** [proposed] Each species MUST have 1 or 2 types; with 2, the order defines primary and secondary.
- **TYP-003** [proposed] The effectiveness chart MUST define a multiplier for every attacker/defender pair.
- **TYP-004** [proposed] Each type MUST list the biomes it is associated with, for encounter weighting.

## Open questions

[Q-008](../open-questions.md#q-008)

## See also

- [Moves](moves.md) · [Battle](../gameplay/battle.md)
