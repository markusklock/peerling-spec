---
title: Catching
type: system
status: draft
req_prefix: CAT
tags: [gameplay, catching, collection]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-3.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-tech-stack-1.md
  - raw/conversations/2026-10-04-decentralize-level-3.md
  - raw/conversations/2026-10-05-proposal-review-1.md
related:
  - wiki/gameplay/battle.md
  - wiki/peerlings/peerling-species.md
updated: 2026-10-05
---

# Catching

> How a player catches a wild Peerling, the catch chance, team size, and what
> happens to a caught Peerling afterwards.

[accepted] Players can catch the Peerlings they encounter.

## Catching without items

[accepted] There are **no items in battles** in the first version, so there are
no catching balls. Catching is a battle action instead: in a wild
battle the player can choose **Catch** instead of a move. It uses the player's
turn, and the wild Peerling then acts as normal. Attempts are unlimited.
[accepted] (approved 2026-10-04)

## Catch chance

[accepted] Approved 2026-10-04:

  chance = 0.6 × (3 × maxHP − 2 × currentHP) ÷ (3 × maxHP) × level factor

- level factor = 1 if the wild Peerling's level is at most the level of the
  player's active Peerling; otherwise max(0.5, 1 − 0.05 × the level difference).
- The roll uses the battle's [random number generator](battle.md#random-number-generator),
  so the server can replay it when verifying the catch.

| Wild Peerling's HP | Chance (same level) |
|--------------------|--------------------:|
| Full | 20% |
| Half | 40% |
| Almost fainted | ≈ 60% |

Why these numbers:
- Since there are no items, the player's only lever is weakening the target, so
  HP matters a lot (3× difference between full and almost fainted).
- It's never a sure thing (max 60%), so attempts stay tense. With unlimited
  attempts, a weakened Peerling still takes about 2 tries on average.
- A failed attempt costs a turn while the wild Peerling keeps attacking, so
  catching is a risk/reward choice.
- It rewards the **quick** move slot ([moves](../peerlings/moves.md#move-slots)):
  its low power is ideal for wearing a Peerling down without knocking it out.
- Catching a Peerling above your level is harder, but never impossible.

A fainted wild Peerling can't be caught; it gives XP
([battle § Experience and levelling](battle.md#experience-and-levelling)).

## Team and collection

[accepted] Approved 2026-10-04:

- **Team size: 4.** Why 4 rather than Pokémon's 6:
  - With 12 types, 4 Peerlings can cover several matchups but not all of them,
    so choosing a team is a real decision.
  - Battles have only 3 moves per Peerling; 4 Peerlings keep PvP battles short
    and readable.
  - In PvP each player downloads the opponent's team models, so fewer is faster.
- **Collection:** every Peerling not in the team. There is no size limit. The
  player can swap Peerlings between team and collection at any time outside
  battle.
- A Peerling caught while the team is full goes to the collection.
- The team is stored in the [save](../tech/player-data.md#save-contents).

[accepted] A caught Peerling becomes a new [instance](../glossary.md#peerling-instance)
in the player's save, referencing its species by CID
([D-0006](../decisions/D-0006-species-vs-instance.md)); its assets are then
retained by the player's node ([NODE-004](../tech/ipfs-helia.md#requirements)).
Caught Peerlings can later be [traded](trading.md).

Creators are notified when their species is caught
([creator-feedback](creator-feedback.md)). Still to be specified: the Peerdex
screen.

[accepted] Any player can verify a catch by replaying the battle from the
catcher's save log ([D-0013](../decisions/D-0013-peer-verified-registry-catches-trades.md)). A catch that fails verification can never
be traded or used in PvP. See
[player-data § Verification](../tech/player-data.md#verification).

## Requirements

- **CAT-001** [accepted] The player MUST be able to catch wild Peerlings.
- **CAT-002** [accepted] A catch MUST record the catch evidence needed to verify it by replay ([SAVE-016](../tech/player-data.md#requirements)).
- **CAT-003** [accepted] There MUST NOT be items in battles in the first version.
- **CAT-004** [accepted] Catching MUST be a battle action that uses the player's turn, available only in wild battles, with unlimited attempts.
- **CAT-005** [accepted] The catch chance MUST follow [Catch chance](#catch-chance).
- **CAT-006** [accepted] A team MUST hold at most 4 Peerlings; all other owned Peerlings are in the collection, which has no size limit.
- **CAT-007** [accepted] The player MUST be able to swap Peerlings between team and collection at any time outside battle.

## Open questions

_None at the moment._
