---
title: "D-0002: All Peerlings are user-generated"
type: decision
status: accepted
tags: [peerlings, content]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-5.md
  - raw/conversations/2026-10-04-answers-round-7.md
related:
  - wiki/peerlings/creation-pipeline.md
updated: 2026-10-04
---

# D-0002: All Peerlings are user-generated

**Status:** accepted (2026-10-03)

## Context
Creature-collecting games normally ship a fixed, hand-designed roster. Peerlings
wants the roster to be the players' collective creation, distributed via IPFS.

## Decision
No hand-designed Peerling species exist when the game launches. Every species is created by a
player through the [creation pipeline](../peerlings/creation-pipeline.md), and
each new player creates one as their starter. (Updated by
[D-0012](D-0012-starter-choice-and-extra-creations.md): new players may instead
choose an existing species, and players can create more later.)

## Consequences
- The roster grows with the player base; the world feels different over time.
- Balance must come from constraints (types, move templates, stat budgets), not
  hand-tuning — see pillar 4 in the [overview](../overview.md).
- Content moderation was expected to be required ([Q-007](../open-questions.md#q-007)).
  Later decided otherwise: no moderation ([D-0010](D-0010-no-content-moderation.md)).
- Cold start: [accepted] the operator creates a handful of
  [seed species](../glossary.md#seed-species) at launch through the same
  pipeline, so the world isn't empty for the first players (resolves
  [Q-020](../open-questions.md#q-020)). These are still player-made
  creations, not a hand-designed roster.

## Alternatives considered
- A hand-made base roster plus user creations — rejected by the designer's brief.
