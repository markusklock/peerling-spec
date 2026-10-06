---
title: Peerling of the Day
type: system
status: accepted
req_prefix: POD
tags: [gameplay, encounters, social, world]
sources:
  - raw/conversations/2026-10-06-v1-fun-features.md
  - raw/conversations/2026-10-06-fun-features-approved.md
  - raw/conversations/2026-10-06-review-2-decisions.md
related:
  - wiki/decisions/D-0022-v1-fun-features.md
  - wiki/gameplay/encounters.md
  - wiki/world/procedural-generation.md
  - wiki/gameplay/world-feed.md
updated: 2026-10-06
---

# Peerling of the Day

> Each day (24 hours) one species is the Peerling of the Day: it appears more
> often everywhere, the world feed announces it, its creator is told, and a
> pedestal near the spawn shows it.

[accepted] Each day, the shared randomness picks one species that
appears more often across the whole world. The world feed announces it and its
creator is notified. No server is needed
([D-0022](../decisions/D-0022-v1-fun-features.md)).

[accepted] The Peerling of the Day is also shown in the world: a pedestal near
the spawn area with its 3D model on top. Hovering over it explains that this
is the Peerling of the Day.

Why: it turns the shared world into a shared event ("have you found today's
Peerling yet?") and gives every creator a chance at a moment in the spotlight.

## Choosing it

[accepted] Details approved 2026-10-06.

- **Day** D = floor(E ÷ 288), where E is the epoch number: a full 24-hour day
  (288 epochs of 5 minutes) from 00:00 to 24:00 UTC. The designer chose a real
  day over the 2-hour in-game day, so each Peerling of the Day is an event that
  lasts long enough for everyone to join.
- The choice uses the [epoch record](../tech/data-formats.md#epoch-record--peerlingsepoch)
  of epoch 288 × D (the server-signed one if it exists, otherwise
  client-derived: [player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)): one species drawn uniformly from
  the eligible species at its `registryHeight`
  ([player-data § Encounter seeds](../tech/player-data.md#encounter-seeds)),
  sorted by registry `seq`, with the
  [random number generator](battle.md#random-number-generator) seeded with
  SHA-256(`"peerlings/spotlight/v1"` ‖ the record's randomness).
- Every client works it out on its own, so all players agree without any
  message.

## In encounters

[accepted] During its day, in every biome, the Peerling of the Day gets the
selection weight max(1, W div 19), where W is the total weight of all other
eligible species ([encounters § Selection](encounters.md#selection)). It is
then about 1 in 20 of all draws, wherever the player is. Its biome and novelty
multipliers don't apply. Encounters use the day of the epoch record they use.

[accepted] If the Peerling of the Day is delisted during its day, its boost
stops (it is no longer eligible at the encounter's registry height,
[ENC-004](encounters.md#requirements)), and the pedestal stays empty until the
next day. A catch's evidence includes the day's record
([data-formats § Catch evidence](../tech/data-formats.md#catch-evidence)).

## In the world

[accepted]
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

[accepted]
- **World feed:** at the start of each day every client adds *"Peerling of
  the Day: Mossnap (by Mia)"* to its own feed ticker. It is worked out
  locally, so nothing is published on the feed topic.
- **Creator:** the creator's client shows *"Your Mossnap is the Peerling of the
  Day!"*. If they were away, it appears in the "since you were last here"
  summary ([creator-feedback](creator-feedback.md#while-away)).
- **Peerdex:** the species' card shows a small sun badge during its day.

## Requirements

- **POD-001** [accepted] Each day (24 hours, from 00:00 UTC), every client MUST derive the same Peerling of the Day from the shared epoch randomness, and that species MUST appear more often in encounters everywhere during that day.
- **POD-002** [accepted] The spawn area MUST have a pedestal showing the Peerling of the Day's 3D model, with a hover explanation.
- **POD-003** [accepted] The world feed MUST announce the Peerling of the Day, and its creator MUST be notified.
- **POD-004** [accepted] The choice, encounter weight, pedestal and announcements MUST follow the rules on this page.

## Open questions

_None at the moment._

## See also

- [Encounters](encounters.md) · [Landmark guardians](guardians.md) · [World feed](world-feed.md)
