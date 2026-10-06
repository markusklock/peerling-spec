---
title: Live World Feed
type: system
status: accepted
req_prefix: FED
tags: [social, pubsub, showcase]
sources:
  - raw/conversations/2026-10-05-showcase-features.md
  - raw/conversations/2026-10-05-phone-backup.md
  - raw/conversations/2026-10-06-review-decisions.md
  - raw/conversations/2026-10-06-v1-fun-features.md
  - raw/conversations/2026-10-06-fun-features-approved.md
  - raw/conversations/2026-10-06-network-performance-approved.md
related:
  - wiki/decisions/D-0016-showcase-features.md
  - wiki/tech/realtime-networking.md
  - wiki/gameplay/creator-feedback.md
updated: 2026-10-06
---

# Live World Feed

> A small ticker of notable events from the whole world, such as new Peerlings
> published and shimmers caught, spread by players over a world-wide pubsub
> topic, with no server needed.

[accepted] The game shows a live feed of notable events across the world
([D-0016](../decisions/D-0016-showcase-features.md)).

## Design

[accepted] Approved 2026-10-05.

- **Topic:** one world-wide pubsub topic, `peerlings/v1/feed`.
- **Events:**

  | Event | Published by | Checked by receivers |
  |-------|-------------|----------------------|
  | *New Peerling published: Lanternfox (by Mia)* | The creator's client, once the registry entry is in | The entry exists in the registry |
  | *Someone caught a shimmer Mossnap!* | The catcher's client | The catch is verified by replay before it's shown, with the light check of [network-performance § Light verification](../tech/network-performance.md#6-light-verification-for-the-world-feed) |
  | *A new Peerling was created at the Creation Shrine* | The creator's client | As for new Peerlings |
  | *Mossnap was first found in the wild by Mia!* [accepted] | The finder's client, once the species stats name them ([creator-feedback § First found in the wild](creator-feedback.md#first-found-in-the-wild)) | The species stats name this catch |
  | *Peerling of the Day: Mossnap (by Mia)* [accepted] | Nobody: each client adds it locally ([peerling-of-the-day](peerling-of-the-day.md#announcements)) | — |
  | *Mia earned all 12 guardian badges!* [accepted] | The player's client ([guardians § Badges](guardians.md#badges)) | All 12 badges verify by replay |

  [accepted] A species created at the Creation Shrine emits only the shrine
  event, not also a "new Peerling published" event.

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
- **FED-002** [accepted] Feed events MUST be published on the world-wide topic `peerlings/v1/feed`, signed by the sender, rate-limited to one per player per minute by receivers, and checked before being shown.
- **FED-003** [accepted] A species created at the Creation Shrine MUST produce only a `shrine-creation` feed event, not a `species-published` event.

## Open questions

_None at the moment._

## See also

- [Creator feedback](creator-feedback.md) · [Realtime networking](../tech/realtime-networking.md)
