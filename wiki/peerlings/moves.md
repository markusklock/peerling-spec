---
title: Moves and Move Templates
type: reference
status: draft
req_prefix: MOV
tags: [peerlings, battle, balance]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/peerlings/types.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/gameplay/battle.md
updated: 2026-10-03
---

# Moves and Move Templates

> Canonical home of the move templates. Every generated attack is a
> [move template](../glossary.md#move-template) filled in with a name,
> description and type by the LLM; the numbers come from the template.

## Why templates

[accepted] Attacks must follow a template base to keep the game balanced. The
LLM is good at *names and flavour* and bad at *numbers*. Templates let it be
creative where it is safe ("Lantern Flare", "Moss Swipe") while all mechanical
values come from a small, hand-balanced table.

## Move structure

[proposed] A generated move contains only:

| Field | From |
|-------|------|
| `template` | LLM chooses a template ID |
| `name` | LLM (moderated, max length TBD) |
| `description` | LLM (one sentence flavor text) |
| `type` | LLM, constrained (see MOV-004) |

All mechanical properties (power, accuracy, uses, effect, priority) are looked up
from the template by the client — they are *not* copied into the species record,
so a template can be rebalanced without republishing species.

## Template table

[proposed] Initial draft; numbers are placeholders until the battle system is
specified ([battle](../gameplay/battle.md)).

| ID | Kind | Power | Accuracy | Effect |
|----|------|-------|----------|--------|
| `strike-basic` | damage | 40 | 100% | — |
| `strike-heavy` | damage | 80 | 80% | — |
| `strike-quick` | damage | 30 | 100% | acts first (priority +1) |
| `strike-drain` | damage | 40 | 100% | heals user for 50% of damage dealt |
| `strike-recoil` | damage | 90 | 100% | user takes 25% of damage dealt |
| `buff-self` | status | — | — | raises one of the user's stats one stage |
| `debuff-foe` | status | — | 95% | lowers one of the foe's stats one stage |
| `heal-self` | status | — | — | restores 40% of max HP; limited uses |

## Requirements

- **MOV-001** [accepted] Every move MUST be an instance of a template in this page's template table.
- **MOV-002** [proposed] Mechanical values MUST come from the template, not from the species record.
- **MOV-003** [proposed] A species' move set MUST contain at least 2 damaging moves.
- **MOV-004** [proposed] A damaging move's type MUST be one of the species' own types or Normal.
- **MOV-005** [proposed] Template IDs MUST be stable; changing a template's numbers is a balance change, recorded as a decision.

## Open questions

[Q-008](../open-questions.md#q-008) · [Q-010](../open-questions.md#q-010) (do Peerlings learn new moves when levelling?)

## See also

- [Types](types.md) · [Battle](../gameplay/battle.md)
