---
title: "D-0012: Starter choice and additional creations"
type: decision
status: accepted
tags: [peerlings, onboarding, creation]
sources:
  - raw/conversations/2026-10-04-answers-round-7.md
  - raw/conversations/2026-10-05-proposal-review-1.md
related:
  - wiki/gameplay/onboarding.md
  - wiki/gameplay/creation-shrine.md
  - wiki/peerlings/creation-pipeline.md
  - wiki/decisions/D-0002-all-peerlings-user-generated.md
updated: 2026-10-05
---

# D-0012: Starter choice and additional creations

**Status:** accepted (2026-10-04, resolves [Q-001](../open-questions.md#q-001));
the balancing details were approved the same day ([Q-035](../open-questions.md#q-035))

## Context
Originally every new player had to create their own starter, and the number of
creations per player was open. Some players may not want to design a creature
before they've even played, and creating is fun enough that players will want
to do it again.

## Decision
- [accepted] A new player either **creates** their starter or **chooses** one
  of a few random existing species instead.
- [accepted] Players can **create additional Peerlings** later. To limit this, a
  place on the map (the [Creation Shrine](../gameplay/creation-shrine.md))
  lets a player give something up, e.g. 3 different Peerlings, in exchange for
  one new creation.
- [accepted] The exact cost and limits are in
  [creation-shrine](../gameplay/creation-shrine.md).

## Consequences
- Not every player is a creator, but every species is still player-made
  ([D-0002](D-0002-all-peerlings-user-generated.md) still holds).
- Starters and shrine creations don't come from a catch, so the server verifies
  them with its own signature instead of a battle replay
  ([player-data § Verification](../tech/player-data.md#verification)).
- GPU load and registry growth depend on the shrine's cost and cooldown.

## Alternatives considered
- One creation per player, ever: simplest, but removes a fun long-term goal.
- Unlimited creations: GPU cost and registry flooding.
