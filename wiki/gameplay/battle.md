---
title: Battle System
type: system
status: stub
req_prefix: BTL
tags: [gameplay, battle, balance]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
related:
  - wiki/peerlings/types.md
  - wiki/peerlings/moves.md
  - wiki/peerlings/peerling-species.md
  - wiki/gameplay/catching.md
  - wiki/gameplay/pvp-battles.md
updated: 2026-10-04
---

# Battle System

> The battle rules, shared by wild battles and [PvP battles](pvp-battles.md),
> and how battles look. Status: stub. Rules are mostly still to be written.

[accepted] Players fight wild Peerlings (Pokémon-inspired) and each other
([D-0008](../decisions/D-0008-shared-multiplayer-world.md)).

## Rules

[proposed] Starting assumptions to confirm: turn-based, one active Peerling per
side, the player can switch Peerlings, use an item, try to catch (wild battles
only), or flee (wild battles only).

Decided: the damage model and turn order below. To be specified: stat stages,
fainting, experience and
rewards ([Q-010](../open-questions.md#q-010)).

### Damage model

[accepted] Approved 2026-10-04 together with the
[stat numbers](../peerlings/peerling-species.md#stat-numbers). It is a
simplified version of the well-known Pokémon formula, adapted to four stats.

- **Levels** run from 1 to 50. PvP uses level 50 for everyone (see
  [pvp-battles](pvp-battles.md)). How XP and levelling work is still open:
  [Q-010](../open-questions.md#q-010).
- **Stats at level L** (all rounded down):
  - HP = 2 × base × L / 100 + L + 10
  - other stats = 2 × base × L / 100 + 5

  At level 50 this gives HP = base + 60 and other stats = base + 5, so the base
  stats can be read directly as level-50 values.
- **Damage** = (((2 × L / 5 + 2) × Power × Attack / Defense) / 50 + 2) ×
  Modifier, rounded down, minimum 1.
- **Modifier** = same-type bonus × type effectiveness × random roll.
  - Same-type bonus: 1.5 when the move's type is one of the user's types,
    otherwise 1.0.
  - Type effectiveness: from [types § Effectiveness chart](../peerlings/types.md#effectiveness-chart).
  - Random roll: uniform from 0.85 to 1.00, using the battle RNG.
- **Turn order:** higher move priority goes first; within the same priority,
  higher Speed goes first; ties are broken by the battle RNG.

With 320 total stats, a typical battle between equal Peerlings at level 50
lasts about 3–6 turns:

| Move (Attack 80 vs Defense 80, same-type bonus) | Damage | Hits to KO a 140 HP Peerling |
|-------------------------------------------------|-------:|---------------------:|
| Quick, power 40, Normal type (no bonus) | ~19 | ~7 |
| Signature, power 60 | ~42 | ~4 |
| Strong, power 100 | ~69 | ~2–3 |
| Strong, power 100, super effective (2×) | ~138 | ~1–2 |

[accepted] The battle engine is **deterministic**: given the starting state,
both sides' actions and the RNG seed, it always gives the same result. PvP
needs this so both players' clients can agree on every turn
([pvp-battles](pvp-battles.md)). The server needs it to verify catches by
replaying the battle ([player-data](../tech/player-data.md#verification),
[D-0009](../decisions/D-0009-player-data-on-orbitdb.md)).

## Presentation

[accepted] Peerlings are static 3D models ([CRE-017](../peerlings/creation-pipeline.md#requirements)),
and battles use simple 3D graphics.

[accepted] Animation is procedural: whole-model transforms, no rigging.
[proposed] Suggested set:

| Moment | Animation |
|--------|-----------|
| Idle | Gentle bob and "breathing" scale pulse; rate can follow the species' temperament |
| Attack | Lunge toward the target and back; squash-and-stretch |
| Hit | Quick shake and a short flash/tint |
| Status / buff | Hop, spin or glow |
| Faint | Tip over and sink or fade out |
| Enter / leave | Pop in/out with scale |

Move visual effects (particles, projectiles, coloured flashes) are generated
from the move's type and template, not authored per move.

## Requirements

- **BTL-001** [accepted] The player MUST be able to battle wild Peerlings they encounter.
- **BTL-002** [accepted] The battle engine MUST be deterministic given the initial state, the actions taken and the RNG seed, on every browser and on the server.
- **BTL-005** [accepted] Damage, stats at a given level and turn order MUST follow the [damage model](#damage-model).
- **BTL-003** [accepted] Battle animation MUST work with static, unrigged models using whole-model transforms only.
- **BTL-004** [proposed] Move visual effects MUST be derived from the move's type and template, so every generated move has an effect without per-move assets.

## Open questions

[Q-008](../open-questions.md#q-008) · [Q-010](../open-questions.md#q-010) ·
[Q-012](../open-questions.md#q-012)

## See also

- [PvP battles](pvp-battles.md) · [Moves](../peerlings/moves.md) · [Types](../peerlings/types.md)
