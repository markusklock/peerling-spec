---
title: Battle System
type: system
status: draft
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
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-05-individual-variation.md
  - raw/conversations/2026-10-05-grid-foliage-battles.md
  - raw/conversations/2026-10-05-pvp-level-modes.md
related:
  - wiki/peerlings/types.md
  - wiki/peerlings/moves.md
  - wiki/peerlings/peerling-species.md
  - wiki/gameplay/catching.md
  - wiki/gameplay/pvp-battles.md
updated: 2026-10-05
---

# Battle System

> The battle rules, shared by wild battles and [PvP battles](pvp-battles.md),
> and how battles look.

[accepted] Players fight wild Peerlings (Pokémon-inspired) and each other
([D-0008](../decisions/D-0008-shared-multiplayer-world.md)).

## Rules

[accepted] There are no items in battles in the first version.

[accepted] The rules below were suggested at the designer's request and
approved 2026-10-05. Guiding idea: classic Pokémon-style
battles, trimmed to what our 3-move, 4-stat, item-free design needs. Every rule
is deterministic apart from rolls of the
[random number generator](#random-number-generator), so battles can be replayed
exactly.

### Formats

| | Wild battle | PvP battle |
|--|-------------|------------|
| Sides | The player's team (up to 4) vs one wild Peerling | Team vs team (up to 4 each) |
| Active Peerlings | One per side | One per side |
| Levels | Real levels | Level 50 for all (Fair mode, default) or real levels, chosen at the challenge ([PVP-010](pvp-battles.md#requirements)) |
| Extra actions | Catch, Flee | none |
| XP | Yes ([Experience and levelling](#experience-and-levelling)) | No |

The first team member that hasn't fainted is sent out first (the *lead*).

### A turn

1. **Choose.** Each side picks one action. In PvP both pick at the same time
   (commit-reveal, [pvp-battles](pvp-battles.md#protocol-proposed)). The wild
   Peerling picks with the rules in [Wild Peerling behaviour](#wild-peerling-behaviour).
   The actions are:
   - **Move:** one of the active Peerling's three moves.
   - **Switch:** swap in another team member that hasn't fainted. This uses the
     turn.
   - **Catch** (wild only): see [catching](catching.md#catch-chance).
   - **Flee** (wild only): always succeeds and ends the battle at once. Being
     forgiving fits a casual game, and fleeing is still a logged encounter.
2. **Resolve, in this order:**
   1. Flee.
   2. Switches (both sides).
   3. Catch attempt. On success the battle ends before the wild Peerling acts.
   4. Moves: higher move priority first (quick moves have priority +1, all
      others 0); within the same priority, higher effective Speed first; ties
      broken by the RNG.
3. **Faints.** A Peerling at 0 HP faints. Its side picks a replacement before
   the next turn; this doesn't use a turn.

### Move mechanics

How each [move template](../peerlings/moves.md#template-table) behaves in battle:
- **Accuracy:** one roll per use of a move; "never misses" skips the roll. A
  miss does nothing.
- **Damage:** the [damage model](#damage-model), once per hit.
- **Two hits** (`quick-flurry`): one accuracy roll, then two hits, each with its
  own damage roll.
- **Recoil** (`strong-recoil`): after hitting, the user loses 33% of the damage
  dealt (at least 1 HP). The user can faint from recoil.
- **Charge** (`strong-charge`): on the first turn the user gathers power and
  does nothing else. On its next turn it strikes automatically; the player
  doesn't pick an action for that turn. If the user is switched out by fainting
  first, the charge is lost.
- **Drain** (`sig-drain`): the user heals 50% of the damage dealt, up to its max
  HP.
- **Stat effects** (`sig-weaken`, `sig-empower`): after a hit, a 30% chance to
  change the chosen stat by one stage (below).

### Stat stages

- Attack, Defense and Speed can be raised or lowered in stages from −3 to +3;
  HP can't. (So the LLM must pick Attack, Defense or Speed for stat-effect
  moves: [moves](../peerlings/moves.md#template-table).)
- Multipliers: −3 → ×0.4, −2 → ×0.5, −1 → ×0.67, 0 → ×1, +1 → ×1.5, +2 → ×2,
  +3 → ×2.5. They apply after level and traits.
- A change beyond ±3 has no effect. Stages reset when a Peerling switches out
  and at the end of the battle.

### Left out on purpose (v1)

- **No critical hits:** the damage roll (0.85–1.00) is enough randomness.
- **No status conditions** (sleep, poison, …): none of the move templates cause
  them, and they'd need more design and UI.

### Wild Peerling behaviour

Each turn the wild Peerling picks a move using the battle RNG:
- 50% of the time, the move with the highest expected damage against the
  player's active Peerling (power × type effectiveness × same-type bonus ×
  accuracy);
- otherwise, one of its three moves at random.

It never flees or switches. This makes it a fair, slightly unpredictable
opponent.

### Ending a battle

| Outcome | Wild battle | PvP battle |
|---------|-------------|------------|
| Opponent's last Peerling faints | Win; XP is awarded | Win |
| Wild Peerling caught | Win; XP is awarded | — |
| Player flees | Battle ends; no XP | — |
| All the player's Peerlings faint | Loss; return to the last rest point, fully healed ([EXP-003](exploration.md#requirements)) | Loss |
| Forfeit / timeout | — | Loss ([pvp-battles](pvp-battles.md)) |

After a battle, HP carries over; stat stages reset.

### PvP turn timer

Each player has 30 s to choose an action. On a timeout, a random move is
chosen for them (via the battle RNG); two timeouts in a row count as a forfeit.

### Battle screen

- **Bottom panel:** three move buttons (name, type color, slot icon). Hovering
  shows power, accuracy, effect, and a hint against the current opponent
  ("Super effective", "Not very effective"). Next to them: Switch, plus Catch and
  Flee in wild battles.
- **Each Peerling:** name, level, type badges, an HP bar with numbers, and stat
  stage arrows when they're not 0. A shimmer sparkles when it enters.
- **Battle text:** "Mossnap used Moss Swipe! It's super effective!"
- **Keyboard:** 1–3 for moves, S switch, C catch, F flee.
- **Pace:** each action animates in about 1.5 s; a toggle doubles the speed.

### Damage model

[accepted] Approved 2026-10-04 together with the
[stat numbers](../peerlings/peerling-species.md#stat-numbers). It is a
simplified version of the well-known Pokémon formula, adapted to four stats.

- **Levels** run from 1 to 50. PvP uses level 50 for everyone in Fair mode (see
  [pvp-battles](pvp-battles.md#fairness)). Levelling: see
  [Experience and levelling](#experience-and-levelling).
- **Stats at level L** (all rounded down):
  - HP = 2 × base × L / 100 + L + 10
  - other stats = 2 × base × L / 100 + 5

  At level 50 this gives HP = base + 60 and other stats = base + 5, so the base
  stats can be read directly as level-50 values.
- **Traits:** [accepted] each stat above is then multiplied by
  (1 + trait ÷ 100), where the trait is the individual's −10…+10 value for that
  stat ([peerling-species § Individual variation](../peerlings/peerling-species.md#individual-variation),
  [D-0014](../decisions/D-0014-individual-variation.md)). Rounded down.
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
- **BTL-008** [accepted] Every stat MUST be multiplied by the individual's trait factor after the level formula.
- **BTL-009** [accepted] Battles MUST follow [Rules](#rules): turn structure and resolution order, move mechanics, stat stages (−3…+3, Attack/Defense/Speed only), wild Peerling behaviour, battle endings and the PvP turn timer.
- **BTL-010** [accepted] There MUST NOT be critical hits or status conditions in v1.

## Open questions

_None at the moment._

## See also

- [PvP battles](pvp-battles.md) · [Moves](../peerlings/moves.md) · [Types](../peerlings/types.md)
