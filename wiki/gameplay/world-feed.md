---
title: Live World Feed
type: system
status: draft
req_prefix: FED
tags: [social, pubsub, showcase]
sources:
  - raw/conversations/2026-10-05-showcase-features.md
related:
  - wiki/decisions/D-0016-showcase-features.md
  - wiki/tech/realtime-networking.md
  - wiki/gameplay/creator-feedback.md
updated: 2026-10-05
---

# Live World Feed

> A small ticker of notable events from the whole world, such as new Peerlings
> published and shimmers caught, spread by players over a world-wide pubsub
> topic, with no server needed.

[accepted] The game shows a live feed of notable events across the world
([D-0016](../decisions/D-0016-showcase-features.md)).

## Design

[proposed] ([Q-044](../open-questions.md#q-044))

- **Topic:** one world-wide pubsub topic, `peerlings/v1/feed`.
- **Events:**

  | Event | Published by | Checked by receivers |
  |-------|-------------|----------------------|
  | *New Peerling published: Lanternfox (by Mia)* | The creator's client, once the registry entry is in | The entry exists in the registry |
  | *Someone caught a shimmer Mossnap!* | The catcher's client | The catch is verified by replay before it's shown ([player-data § Catches](../tech/player-data.md#catches-accepted-details-proposed)) |
  | *A new Peerling was created at the Creation Shrine* | The creator's client | As for new Peerlings |

- **Signed and limited.** Every feed message is signed by the sender's identity
  key. Receivers show at most one message per player per minute, and ignore
  flagged players ([player-data § Transfer log and trades](../tech/player-data.md#transfer-log-and-trades)).
  Unverifiable messages are dropped.
- **Display:** a small ticker in a corner of the screen showing the last few
  events. Clicking an event opens the Peerling card ([sharing](sharing.md)).
- **Side effect:** checking a shimmer catch fetches the catcher's save log, which
  also adds to the peer save backups
  ([player-data § Keeping saves available](../tech/player-data.md#keeping-saves-available)).

## Requirements

- **FED-001** [accepted] The game MUST show a live feed of notable world events.
- **FED-002** [proposed] Feed events MUST be published on the world-wide topic `peerlings/v1/feed`, signed by the sender, rate-limited to one per player per minute by receivers, and checked before being shown.

## Open questions

[Q-044](../open-questions.md#q-044)

## See also

- [Creator feedback](creator-feedback.md) · [Realtime networking](../tech/realtime-networking.md)
