---
title: Peerling of the Day
type: system
status: draft
req_prefix: POD
tags: [gameplay, encounters, social, world]
sources:
  - raw/conversations/2026-10-06-v1-fun-features.md
related:
  - wiki/decisions/D-0022-v1-fun-features.md
  - wiki/gameplay/encounters.md
  - wiki/world/procedural-generation.md
  - wiki/gameplay/world-feed.md
updated: 2026-10-06
---

# Peerling of the Day

> Each in-game day one species is the Peerling of the Day: it appears more
> often everywhere, the world feed announces it, its creator is told, and a
> pedestal near the spawn shows it.

[accepted] Each in-game day, the shared randomness picks one species that
appears more often across the whole world. The world feed announces it and its
creator is notified. No server is needed
([D-0022](../decisions/D-0022-v1-fun-features.md)).

[accepted] The Peerling of the Day is also shown in the world: a pedestal near
the spawn area with its 3D model on top. Hovering over it explains that this
is the Peerling of the Day.

Why: it turns the shared world into a shared event ("have you found today's
Peerling yet?") and gives every creator a chance at a moment in the spotlight.

## Choosing it

[proposed] Details below await approval ([Q-055](../open-questions.md#q-055)).

- **Day** D = floor(E ÷ 24), where E is the epoch number. This is the in-game
  day of [procedural-generation § Day and night](../world/procedural-generation.md#day-and-night)
  (2 hours, starting at in-game midnight).
- The choice uses the [epoch record](../tech/data-formats.md#epoch-record--peerlingsepoch)
  of epoch 24 × D (signed or client-derived): one species drawn uniformly from
  the eligible species at its `registryHeight`
  ([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)),
  sorted by registry `seq`, with the
  [random number generator](battle.md#random-number-generator) seeded with
  SHA-256(`"peerlings/spotlight/v1"` ‖ the record's randomness).
- Every client works it out on its own, so all players agree without any
  message.

## In encounters

[proposed] During its day, in every biome, the Peerling of the Day gets the
selection weight max(1, W div 19), where W is the total weight of all other
eligible species ([encounters § Selection](encounters.md#selection)). It is
then about 1 in 20 of all draws, wherever the player is. Its biome and novelty
multipliers don't apply. Encounters use the day of the epoch record they use.

## In the world

[proposed]
- **Pedestal:** in the [spawn hub](../world/procedural-generation.md#spawn-hub),
  next to the New Peerlings gallery: a larger raised pedestal with a sun
  emblem and a soft light beam visible from afar. The species' 3D model stands
  on top, slowly turning, loaded live from IPFS, and shimmers now and then.
- **Hover:** a tooltip says *"Peerling of the Day: Mossnap, created by Mia.
  Appears more often everywhere in the world for another 1 h 12 min."*
- **Click:** opens the species card ([sharing](sharing.md#peerling-card)).
- At the start of each day the pedestal swaps to the new species with a short
  glow effect.

## Announcements

[proposed]
- **World feed:** at the start of each day every client adds *"Peerling of
  the Day: Mossnap (by Mia)"* to its own feed ticker. It is worked out
  locally, so nothing is published on the feed topic.
- **Creator:** the creator's client shows *"Your Mossnap is the Peerling of the
  Day!"*. If they were away, it appears in the "since you were last here"
  summary ([creator-feedback](creator-feedback.md#while-away)).
- **Peerdex:** the species' card shows a small sun badge during its day.

## Requirements

- **POD-001** [accepted] Each in-game day, every client MUST derive the same Peerling of the Day from the shared epoch randomness, and that species MUST appear more often in encounters everywhere during that day.
- **POD-002** [accepted] The spawn area MUST have a pedestal showing the Peerling of the Day's 3D model, with a hover explanation.
- **POD-003** [accepted] The world feed MUST announce the Peerling of the Day, and its creator MUST be notified.
- **POD-004** [proposed] The choice, encounter weight, pedestal and announcements MUST follow the rules on this page.

## Open questions

- [Q-055](../open-questions.md#q-055) — approve the exact rules for the v1 fun features

## See also

- [Encounters](encounters.md) · [Landmark guardians](guardians.md) · [World feed](world-feed.md)
