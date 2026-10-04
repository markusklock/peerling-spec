---
title: Moves and Move Templates
type: reference
status: draft
req_prefix: MOV
tags: [peerlings, battle, balance]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-2.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
related:
  - wiki/peerlings/types.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/peerlings/peerling-species.md
  - wiki/gameplay/battle.md
updated: 2026-10-04
---

# Moves and Move Templates

> Canonical home of move slots, move templates and move-set rules. Every
> generated attack fills a [move slot](../glossary.md#move-slot) using a
> [move template](../glossary.md#move-template). The LLM provides the name,
> description and type; the numbers come from the template.

## Why templates

[accepted] Attacks must follow a template base to keep the game balanced. The
LLM is good at *names and flavour* and bad at *numbers*. Templates let it be
creative where it is safe ("Lantern Flare", "Moss Swipe"), while every
mechanical value comes from a small, hand-balanced table.

## Move slots

[accepted] Every species has exactly **three** moves, one per slot: a quick
attack, a strong attack and a special (signature) attack. There is no fourth
slot. [proposed] One move per slot gives every Peerling a comparable toolkit. Identity comes from type, stat
spread and the signature move's flavour, not from having better moves.

[proposed] Slot roles:

| Slot | Role | Trade-off | Allowed move type |
|------|------|-----------|-------------------|
| **Quick** | Reliable, acts first | Low power | Normal or one of the species' types |
| **Strong** | Big hit | Each template has a drawback (misses, recoil, charge turn) | Normal or one of the species' types |
| **Signature** | The creature's special move, the one players remember | Medium power plus a secondary effect | The species' primary type |

"Signature" is the designer's "special attack". The word "special" is avoided as
a slot name because Pokémon uses it for a damage category and a stat.

## Move structure

[proposed] A generated move in the species record contains only:

| Field | From |
|-------|------|
| `slot` | Fixed by the slot being filled |
| `template` | LLM chooses a template ID allowed for that slot |
| `name` | LLM (max length TBD) |
| `description` | LLM (one sentence of flavour text) |
| `type` | LLM, constrained by the slot (table above) |

[accepted] A Peerling's moves are fixed when its species is created and
**never change**: no learning new moves on level-up, no move tutors.

[accepted] Moves can be used **without limit**: there are no per-battle uses
(PP) and no stamina. Balance comes from the trade-offs built into each
template (low power, misses, recoil, charge turns).

All mechanical properties (power, accuracy, priority, effect) are looked up
from the template by the client. They are *not* copied into the species record,
so a template can be rebalanced without republishing species.

## Template table

[proposed] First draft. The numbers are placeholders until the damage formula is
specified in [battle](../gameplay/battle.md).

| ID | Slot | Power | Accuracy | Priority | Effect |
|----|------|-------|----------|----------|--------|
| `quick-jab` | quick | 40 | 100% | +1 | — |
| `quick-flurry` | quick | 2 × 20 | 100% | +1 | hits twice |
| `strong-heavy` | strong | 100 | 75% | 0 | — |
| `strong-recoil` | strong | 100 | 100% | 0 | user takes 33% of damage dealt |
| `strong-charge` | strong | 130 | 100% | 0 | charges on the first turn, hits on the second |
| `sig-drain` | signature | 60 | 100% | 0 | heals user for 50% of damage dealt |
| `sig-weaken` | signature | 60 | 100% | 0 | 30% chance to lower one foe stat one stage |
| `sig-empower` | signature | 60 | 100% | 0 | 30% chance to raise one own stat one stage |
| `sig-sure` | signature | 70 | never misses | 0 | — |

For the `sig-weaken` and `sig-empower` templates, the LLM also chooses which
stat is affected. That choice is stored in the move entry as `stat`.

## Reference: how Pokémon-like games structure moves

Researched to answer the designer's question (2026-10-03). This is background,
not a decision.

- **Pokémon.** Each move has a type, a category (physical, special or
  status), power, accuracy, PP (number of uses), priority and an optional
  secondary effect. A Pokémon knows at most 4 moves at a time, chosen from its
  species' *learnset* (moves learned by level, from TMs or from breeding).
  Most species mainly have moves of their own type, because those get a
  same-type attack bonus (STAB, 1.5× damage). They round this out with Normal
  moves and a few *coverage* moves of other types to hit their weaknesses.
  Balance comes mostly from **stat distribution** (base-stat totals range
  roughly from 180 to 720) and **which moves a species can access**, both
  hand-tuned per species.
- **Temtem.** Techniques cost *stamina* instead of PP, and the strongest ones
  need *hold* turns to charge. The power/tempo trade-off is built into each
  move.
- **Cassette Beasts.** Moves cost from a shared pool of action points (AP),
  so stronger moves mean fewer actions.

**Takeaways for Peerlings.** Pokémon's approach depends on hand-tuning every
species' learnset and stat total, which user-generated content can't have.
Fixed role slots plus templates with built-in trade-offs (the Temtem idea)
keep every Peerling comparable. The equal stat total
([peerling-species § Stats](peerling-species.md#stats)) does the same for
stats. A same-type bonus is a good way to make the type matter; it belongs in
the damage formula ([battle](../gameplay/battle.md)).

## Requirements

- **MOV-001** [accepted] Every move MUST be an instance of a template in this page's template table.
- **MOV-002** [proposed] Mechanical values MUST come from the template, not from the species record.
- ~~**MOV-003**~~ (removed 2026-10-03, replaced by the slot rules MOV-006 and MOV-008)
- ~~**MOV-004**~~ (removed 2026-10-03, replaced by the per-slot type rule MOV-007)
- **MOV-005** [proposed] Template IDs MUST be stable; changing a template's numbers is a balance change, recorded as a decision.
- **MOV-006** [accepted] A species' move set MUST contain exactly three moves, one for each slot: quick, strong, signature.
- **MOV-007** [proposed] Quick and strong moves MUST be Normal type or one of the species' types; the signature move MUST be the species' primary type.
- **MOV-008** [proposed] A move's template MUST be one allowed for its slot.
- **MOV-009** [accepted] Moves MUST be usable without limit; there MUST NOT be per-battle use counts or a move resource.
- **MOV-010** [accepted] A Peerling's move set MUST NOT change after its species is published.

## Open questions

_None at the moment._

## See also

- [Types](types.md) · [Battle](../gameplay/battle.md)
