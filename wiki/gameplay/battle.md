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
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-5.md
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

[accepted] There are no items in battles in the first version.

[proposed] Turn-based, one active Peerling per side. Each turn the player
either uses one of the active Peerling's three moves, switches to another team
member (team size: [catching](catching.md#team-and-collection)), tries to
[catch](catching.md#catching-without-items) (wild battles only), or flees (wild
battles only).

Decided: the damage model, turn order and experience below. To be specified:
stat stages, fainting, healing between battles
([Q-032](../open-questions.md#q-032)), and rewards other than XP.

### Damage model

[accepted] Approved 2026-10-04 together with the
[stat numbers](../peerlings/peerling-species.md#stat-numbers). It is a
simplified version of the well-known Pokémon formula, adapted to four stats.

- **Levels** run from 1 to 50. PvP uses level 50 for everyone (see
  [pvp-battles](pvp-battles.md)). Levelling: see
  [Experience and levelling](#experience-and-levelling).
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

### Experience and levelling

[accepted] A simple XP curve; no evolution
([SPC-011](../peerlings/peerling-species.md#requirements)); moves never change
([MOV-010](../peerlings/moves.md#requirements)).

[accepted] Numbers (approved 2026-10-04):

| Rule | Value |
|------|-------|
| XP needed to go from level L to L + 1 | 5 × L² |
| XP gained for each wild Peerling defeated **or caught** | 10 × the wild Peerling's level |
| Who gets the XP | Every one of the player's Peerlings that took part in the battle and didn't faint, each gets the full amount |
| XP from PvP | None ([pvp-battles § Rewards](pvp-battles.md#rewards)) |
| Starter level | 5 |
| Maximum level | 50; XP stops accumulating there |
| On level-up | Stats are recomputed with the stat formula above; current HP rises by the same amount as max HP |

Pacing: fighting wild Peerlings of about its own level, a Peerling needs about
L ÷ 2 battles per level. Going from level 5 to 50 takes about 600 wild battles
(roughly 10 hours at a minute per battle). Wild levels come from the distance to
the world centre ([encounters § Wild level](encounters.md#wild-level)), so
players level up by exploring further out.

### Random number generator

[proposed] Every random roll in a battle (accuracy, damage roll, secondary
effects, catch chance, speed ties) and in encounter selection comes from one
deterministic generator, so a battle can be replayed exactly on any browser and
on the server ([BTL-002](#requirements)):

- Input: a 32-byte seed.
- Output block *i* (i = 0, 1, 2, …) = SHA-256(seed ‖ i as 8-byte big-endian).
- Each block yields eight unsigned 32-bit integers (big-endian), consumed in
  order. A uniform value in [0, 1) is the next integer ÷ 2³².
- An integer in [0, n) is drawn by rejection sampling (discard values ≥ the
  largest multiple of n below 2³²), so there is no bias.
- Rolls are drawn in a fixed order that the battle rules define.

Seeds: wild battles use the [encounter seed](../tech/player-data.md#encounter-seeds);
PvP battles use the commit-reveal seed ([pvp-battles](pvp-battles.md)).

## Presentation

[accepted] The visual style is colorful ([visual-style](../world/visual-style.md)).
[proposed] Battles take place on the spot in the 3D world; the battle camera is
defined in [visual-style § Camera](../world/visual-style.md#camera).

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
- **BTL-003** [accepted] Battle animation MUST work with static, unrigged models using whole-model transforms only.
- **BTL-004** [proposed] Move visual effects MUST be derived from the move's type and template, so every generated move has an effect without per-move assets.
- **BTL-005** [accepted] Damage, stats at a given level and turn order MUST follow the [damage model](#damage-model).
- **BTL-006** [accepted] Levels MUST run from 1 to 50, and XP gain and the XP curve MUST follow [Experience and levelling](#experience-and-levelling).
- **BTL-007** [proposed] All randomness in battles and encounter selection MUST come from the [random number generator](#random-number-generator) defined above.

## Open questions

[Q-032](../open-questions.md#q-032)

## See also

- [PvP battles](pvp-battles.md) · [Moves](../peerlings/moves.md) · [Types](../peerlings/types.md)
