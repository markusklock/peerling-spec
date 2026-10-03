---
title: "D-0002: All Peerlings are user-generated"
type: decision
status: accepted
tags: [peerlings, content]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
related:
  - wiki/peerlings/creation-pipeline.md
updated: 2026-10-03
---

# D-0002: All Peerlings are user-generated

**Status:** accepted (2026-10-03)

## Context
Creature-collecting games normally ship a fixed, hand-designed roster. Peerlings
wants the roster to be the players' collective creation, distributed via IPFS.

## Decision
No Peerling species exist when the game launches. Every species is created by a
player through the [creation pipeline](../peerlings/creation-pipeline.md), and
each new player creates one as their starter.

## Consequences
- The roster grows with the player base; the world feels different over time.
- Balance must come from constraints (types, move templates, stat budgets), not
  hand-tuning — see pillar 4 in the [overview](../overview.md).
- Content moderation is required ([Q-007](../open-questions.md#q-007)).
- Cold start must be handled ([Q-020](../open-questions.md#q-020)).

## Alternatives considered
- A hand-made base roster plus user creations — rejected by the designer's brief.
