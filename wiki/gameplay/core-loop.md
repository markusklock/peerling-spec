---
title: Core Gameplay Loop
type: overview
status: accepted
tags: [gameplay, loop]
sources:
  - raw/conversations/2026-10-03-initial-vision.md
  - raw/conversations/2026-10-03-answers-round-1.md
  - raw/conversations/2026-10-04-answers-round-4.md
  - raw/conversations/2026-10-05-proposal-review-2.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-v1-fun-features.md
related:
  - wiki/gameplay/onboarding.md
  - wiki/gameplay/exploration.md
  - wiki/gameplay/encounters.md
  - wiki/gameplay/battle.md
  - wiki/gameplay/catching.md
updated: 2026-10-06
---

# Core Gameplay Loop

> What the player does, minute to minute and session to session. Each step links
> to its canonical page.

## One-time: onboarding

[accepted] Every new player creates their [player character](player-character.md)
and creates or chooses their starting Peerling. See [onboarding](onboarding.md).

## The loop

[accepted] Travel around a procedurally generated world looking for Peerlings to
fight and catch.

```mermaid
flowchart LR
  E[Explore the world] --> N[Encounter a wild Peerling]
  N --> B[Battle]
  B -->|weakened| C[Try to catch]
  B -->|won / fled| E
  C -->|caught| T[Add to collection / team]
  C -->|escaped| B
  T --> E
```

| Step | Canonical page | Provenance |
|------|----------------|------------|
| Explore | [exploration](exploration.md) | [accepted] |
| Encounter | [encounters](encounters.md) | [accepted] |
| Battle | [battle](battle.md) | [accepted] |
| Catch | [catching](catching.md) | [accepted] |

## The social layer

[accepted] The world is shared ([D-0008](../decisions/D-0008-shared-multiplayer-world.md)).
Around the core loop, players meet each other, [battle](pvp-battles.md) and
[trade](trading.md). See [multiplayer](multiplayer.md).

## Motivations

[accepted] Why players keep playing:
- **Discovery** — every wild Peerling is another player's imagination; there is
  always something never seen before.
- **Collection** — catch as many different species as possible (the [Peerdex](peerdex.md)
  of everything you have seen and caught).
- **Mastery** — build a team that handles every type matchup; reach harder
  areas further from the start; earn the 12 [guardian](guardians.md) badges;
  beat other players.
- **Discovery race** — be the first to find a new Peerling in the wild
  ([creator-feedback § First found in the wild](creator-feedback.md#first-found-in-the-wild)),
  or hunt today's [Peerling of the Day](peerling-of-the-day.md).
- **Exchange** — trade to get species you can't find yourself, or to spread
  your own creation.
- **Pride** — see how your own creation spreads through the world
  ([creator-feedback](creator-feedback.md)).

## Open questions

_None at the moment._
